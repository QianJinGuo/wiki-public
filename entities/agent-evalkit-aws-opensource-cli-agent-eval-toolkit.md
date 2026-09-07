---
title: "Agent-EvalKit：AWS 开源 CLI Agent 评测工具包"
created: 2026-06-12
updated: 2026-09-07
type: entity
tags: [agent, evaluation, aws, opensource, cli, claude-code, opentelemetry, agentcore, bedrock, observability, testing, faithfulness]
sources: [raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit]
review_value: 7
review_confidence: 8
summary: AWS 开源 (Apache 2.0) Agent 评测 CLI 工具包，通过 AI 编码助手（Claude Code/Kiro CLI/Kilo Code）作为评测引擎，整合 6 阶段流水线（Plan/Data/Trace/Run/Eval/Report），用 OpenTelemetry 追踪执行路径，LLM-as-judge + 第三方库（DeepEval/Strands Evals）评估，输出代码级修复建议。旅行研究案例暴露 32.3% Faithfulness 虚假率。
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent-EvalKit：AWS 开源 CLI Agent 评测工具包

> 原文存档：[[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit|原文存档]]

AWS 2026-06-11 开源（Apache 2.0）的 Agent 评测工具包，定位是**"用 AI 编码助手本身做评测引擎"**——Claude Code / Kiro CLI / Kilo Code 通过 slash command (`/evalkit.*`) 直接驱动整个评测流程，无需独立评测平台。该工具包在 [GitHub awslabs/Agent-EvalKit](https://github.com/awslabs/Agent-EvalKit) 维护。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 核心问题

传统 Agent 评测只看最终输出是否匹配期望，但 Agent 行为有以下隐藏失败模式： ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

- **幻觉性输出**：Agent 返回结构良好、可读性高的响应，但工具返回空结果时，Agent 静默编造数据并呈现为真实查询结果 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **流程性失败**：Agent 跳过了可靠性所需的验证步骤 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **工具调用错位**：Agent 用了正确的工具但传错参数 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

仅靠输出层断言无法捕获这些"最终响应下方的失败"，必须**追踪完整执行路径**（工具调用、数据返回、响应忠实性）才能诊断。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 关键设计：CLI 形式 + AI 编码助手即引擎

Agent-EvalKit 区别于现有 Agent 评测框架（LangSmith、AgentEval、阿里云泊予）的核心机制： ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

1. **不是独立平台**——Claude Code / Kiro CLI / Kilo Code 本身是评测引擎，通过 `/evalkit.*` slash command 驱动 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
2. **自然语言描述评测目标**——`/evalkit.plan` 后接 "Evaluate my agent for response quality and tool accuracy"，工具包根据指引生成对应评测代码 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
3. **六阶段产物在 `eval/` 目录中串联**——每个阶段读取前阶段产物生成下一阶段 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
4. **每阶段可独立重跑 + 调整指引**——无需重建 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 六阶段流水线

| 阶段 | Slash Command | 产物 | 关键能力 |
|------|---------------|------|----------|
| **Plan** | `/evalkit.plan` | `eval/plan.json` | 读 agent 源码 + 工具定义 + system prompt，生成针对该 agent 的评测方案（每个 metric 配具体评测方法） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Data** | `/evalkit.data` | `eval/test_cases.json` | 根据 plan 生成 ground-truth 测试集（多轮、覆盖能力 + 失败模式）；可指向现有生产日志数据集 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Trace** | `/evalkit.trace` | OTel instrumentation | 加 OpenTelemetry 兼容追踪；自动检测 Strands/LangGraph/CrewAI 框架并应用对应 instrumentation ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Run agent** | `/evalkit.run_agent` | `eval/traces/*.json` | 跑每个 test case 收集结构化 trace（工具调用、模型响应、中间状态） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Eval** | `/evalkit.eval` | `eval/results.json` | 把 plan 里的 metric 编译为可执行评测代码；支持 DeepEval + Strands Evals SDK 等库 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Report** | `/evalkit.report` | `eval/report.md` | 跨 test case 模式分析 + 优先级排序 + 引用**代码特定位置**的修复建议（每条建议含预期影响） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |

## 三个核心评测维度

| 维度 | 衡量什么 | 失败模式 |
|------|----------|----------|
| **Faithfulness（忠实性）** | 响应是否真实反映工具返回数据 | 工具返回空结果时，Agent 静默编造数据（最严重的隐藏失败） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Tool Parameter Accuracy** | Agent 是否以正确参数调用正确工具 | 参数精度问题（如传错城市代码、错误日期格式） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |
| **Response Quality** | 输出连贯性、用户可读性 | 形式良好但内容空泛 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |

## 案例实证：旅行研究 Agent 评测结果

Agent-EvalKit 团队在 Strands Agents SDK + Bedrock 旅行研究 agent 上跑 100 轮多轮评测 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]：

| 维度 | 分数 | 解读 |
|------|------|------|
| Response Quality | 83.9% | 输出连贯、可读性高 — **表面看起来工作正常** |
| Tool Parameter Accuracy | 64.5% | 工具选择大体正确，但参数精度有问题 |
| **Faithfulness** | **32.3%** | **严重幻觉** — 工具返回空/不完整结果时编造汇率、温度、景点信息 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md] |

**关键洞察**：仅看 Response Quality（83.9%），团队会以为 agent "工作良好"；但 Faithfulness 32.3% 揭示了**最终响应下方隐藏的可靠性失败**。Report 阶段生成的最优先建议：system prompt 加 "工具返回空时显式披露" 指令 + 全代码路径工具错误处理改进 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]。

## 与现有评测框架的差异化

| 框架 | 定位 | 与 Agent-EvalKit 的差异 |
|------|------|---------------------|
| **AgentEval（wallezhang，YAML 驱动）** | Go + YAML 配置，pass@k/pass^k 指标 + CI/CD 集成 | AgentEval 是**评测配置 + 指标体系**框架；Agent-EvalKit 是**AI 编码助手驱动的 6 阶段流水线**——AgentEval 缺自动执行路径追踪 |
| **阿里云泊予 Harness 式评测** | Claude Code 作 Harness 搭建者，把 test_runner.py 替换为评测 Agent 提示词 | 泊予侧重**业务 Agent 评测工程范式**（Harness 提示词替代 Python 脚本）；Agent-EvalKit 侧重**6 阶段流水线 + 自动 test case 生成 + 自动代码级建议**——泊予是手动构建评测，Agent-EvalKit 是 AI 自动生成 |
| **LangSmith Evaluation Concepts** | conceptual 概览：拆解 agent 为 output/retrieval/tool/trajectory + offline vs online | LangSmith 是概念框架；Agent-EvalKit 是**可立即 install + 跑的 OSS 工具**（`uv tool install evalkit`） |
| **OpenTelemetry agent instrumentation**（通用） | 通用 trace 协议 | Agent-EvalKit **自动应用** OTel 兼容追踪到 Strands/LangGraph/CrewAI 框架 |

**核心差异化**：Agent-EvalKit 是**第一个**把"AI 编码助手作为评测引擎 + 6 阶段自动流水线 + 代码级修复建议"三者整合的开源工具包。竞争对手要么是概念层（LangSmith），要么是配置层（AgentEval），要么是工程范式层（泊予）——但都没有把"AI 自动生成 test case + AI 写评测代码 + AI 输出代码级修复建议"做成端到端可执行流水线。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 工程实践要点

- **安装**：`uv tool install evalkit --from git+https://github.com/awslabs/Agent-EvalKit.git` ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **前置**：AWS account + Bedrock 启用 foundation model + Python 3.11+ + uv + Claude Code/Kiro CLI/Kilo Code ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **支持的 agent 框架**：Strands Agents SDK、LangGraph、CrewAI（Trace 阶段自动检测） ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **支持评测库**：DeepEval + Strands Evals SDK ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
- **最佳实践**：从小范围开始（2-3 个 metric）→ 用领域知识驱动 natural language 指引 → 集成到 CI/CD ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 实践启示

1. **AI 编码助手是评测新基础设施**：用 Claude Code 等 AI 助手驱动评测流水线，无需团队自建 evaluation 平台 — 这是评测领域的"工具翻转" ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
2. **Faithfulness 比 Response Quality 关键 2.6x**：旅行 agent 案例中，83.9% 的输出质量掩盖了 32.3% 的忠实性失败 — **单一指标评测会误导**，必须多维度联合评估 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
3. **代码级建议优于数字仪表盘**：Report 阶段输出"具体代码位置 + 预期影响"是 Agent-EvalKit 的最大价值 — 把评测分数转化为可执行修复 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
4. **Apache 2.0 协议 + CLI 形式**：降低准入门槛到 `uv tool install`，team 可以立即在没有 LangSmith 商业 license 的情况下获得 production-grade agent eval ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 相关实体

- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework]] — YAML 驱动的 AgentEval 框架，Go 实现
- [[entities/harness-engineered-business-agent-evaluation-aliyun-boyu]] — 阿里云泊予用 Claude Code 搭建评测 Harness
- [[entities/langsmith-evaluation-concepts]] — LangSmith 评测概念框架
- [[entities/agent-harness-observability-production]] — Agent harness 生产可观测性
- [[entities/agent-memory-evaluation-landscape-taobao-survey]] — Agent 记忆评估方法论全景
- [[entities/aws-cn-intelligent-device-assistant-consumer-agent-2026|基于 aws 智能设备助手行业资产，构建社交渠道触达的消费级 agent 交互应用]]
- [[entities/使用-aws-security-agent-构建应用安全闭环从代码提交到漏洞修复的自动化之路|使用 aws security agent 构建应用安全闭环：从代码提交到漏洞修复的自动化之路]]

- [[entities/browser-request-recording-ai-code-generation-e2e-api-testing|基于浏览器请求录制与ai代码生成的e2e接口自动化测试实践]]

- [[moc/evaluation-and-benchmarks|MOC]]
## 深度分析

### 核心观点：AI 编码助手作为评测引擎是新范式

Agent-EvalKit 的核心创新不是某个算法或指标，而是一个**范式转移**：用 AI 编码助手本身驱动评测流水线。传统 Agent 评测需要团队自建 evaluation 平台、维护评测基础设施；而 Agent-EvalKit 把这个负担转移给 Claude Code/Kiro CLI/Kilo Code——已经是团队标准工具的 AI 编码助手。这是评测领域的"工具翻转"：工具不再是被评测的对象，而是驱动评测过程的主体。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 技术要点：Faithfulness 是隐藏的致命失败模式

旅行研究 Agent 案例揭示了 **Faithfulness（忠实性）失败**的特殊性：当工具返回空结果时，Agent 静默编造汇率、温度、景点信息——这些编造数据在 Response Quality 评测中完全不会被发现，因为 Response Quality 只看输出"形式是否良好、可读性是否高"。Faithfulness 32.3% 与 Response Quality 83.9% 之间的巨大差距说明**单一输出质量指标会系统性地掩盖 agent 的可靠性缺陷**。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 实践价值：六阶段流水线覆盖完整评测生命周期

Agent-EvalKit 的 Plan → Data → Trace → Run → Eval → Report 六阶段流水线对应了 agent 评测的完整生命周期：Plan 阶段根据 agent 源码和工具定义生成针对性评测方案；Data 阶段生成覆盖能力 + 失败模式的测试集；Trace 阶段用 OpenTelemetry 追踪执行路径；Eval 阶段把 metric 编译为可执行评测代码；Report 阶段输出代码级修复建议。这个流水线设计使**每阶段可独立重跑 + 调整指引**，无需重建整个评测。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 核心观点：OTel 追踪使行为诊断从"不可能"变为"可操作"

传统输出层断言无法区分"工具返回空结果时 agent 正确披露"和"工具返回空结果时 agent 静默编造"——这两种情况输出形式完全相同。Agent-EvalKit 通过 OpenTelemetry 追踪**完整执行路径**（工具调用、数据返回、响应忠实性），使这种区分成为可能。这是评测基础设施层面的关键能力提升。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 技术要点：LLM-as-judge 需要第三方评测库补充

Eval 阶段使用 LLM-as-judge + DeepEval + Strands Evals SDK 的组合，说明**单一 LLM-as-judge 不足以覆盖所有评测维度**。DeepEval 等第三方库提供针对特定失败模式的专项检测（如 hallucination、tool call accuracy），而 LLM-as-judge 擅长评估开放式 response quality。两者配合才能实现完整的多维度评测覆盖。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

## 实践启示

### 1. 从小范围开始，优先评测 Faithfulness 而非 Response Quality

最佳实践是从 2-3 个 metric 开始，先跑小范围评测验证 pipeline 可用性。在选择 metric 优先级时，**优先评测 Faithfulness**——因为它是隐藏最深的失败模式，Response Quality 分数高不代表 agent 可靠。用领域知识驱动 natural language 指引描述评测目标，比预设指标更能捕获针对性问题。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 2. 将评测集成到 CI/CD，用代码级修复建议驱动开发闭环

Agent-EvalKit 的 Report 阶段输出"具体代码位置 + 预期影响"修复建议，这是把评测分数转化为可执行开发任务的关键。团队应该将评测结果直接接入 CI/CD：每次代码变更跑评测 → 发现 Faithfulness 下降 → Report 生成修复建议 → 开发者按建议修改代码。这是 **评测驱动的 agent 可靠性工程闭环**。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 3. 充分利用 OTel 框架的自动检测能力

Trace 阶段自动检测 Strands/LangGraph/CrewAI 框架并应用对应 instrumentation，这是降低接入成本的关键。团队不需要手动为每个框架配置追踪——只需要跑 `/evalkit.trace`，工具包自动应用 OTel 兼容追踪。配合 "Agent 可观测性" 的最佳实践，可以建立完整的 agent 行为可观测性。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 4. Apache 2.0 + CLI 形式使评测门槛降到最低

`uv tool install evalkit` 即可安装，无需 LangSmith 商业 license 或自建评测平台。对于预算有限的团队或早期阶段的 agent 项目，Agent-EvalKit 提供了**生产级别的评测能力而不需要商业投入**。这是开源工具对 AI agent 工程生态的重要贡献。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

### 5. 结合 "Agent 评估基准体系" 设计评测策略

Agent-EvalKit 擅长针对特定 agent 生成定制化评测方案，但团队仍需要一套系统的评测策略框架。参见 "Agent 评估基准体系" 了解评测维度的理论背景，结合 agent 的具体能力边界设计有针对性的评测方案。 ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]

→ [[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit|原文存档]] ^[raw/articles/evaluate-ai-agents-systematically-with-agent-evalkit.md]
