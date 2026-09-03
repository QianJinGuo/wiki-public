---

title: "Harness 之后：Agent 可靠性的关键，是状态边界和失败闭环"
created: 2026-06-10
updated: 2026-08-01
sources: [raw/articles/harness-之后-状态边界与失败闭环-若飞, raw/articles/vivo-agent-react-to-harness-state-schema-2026-07-22]
tags: [agent, architecture, code, data, database, evaluation, fine-tuning, game, harness-engineering, llm, memory, mlops, observability, prompt, search, security, tool-use, workflow, state-schema, runtime-fact]
review_value: 7
review_confidence: 7
type: entity
---

# Harness 之后：Agent 可靠性的关键，是状态边界和失败闭环


## 相关实体

- [[entities/ai-native-startup-cyberfund-2026|how to build an ai-native startup]]
- [[entities/flashlabs-vertical-ai-startup-pivot|垂类 ai 创企的自救：flashlabs 从 flashintel 到 ai native]]
- [[entities/latest-open-artifacts-19-qwen-35-glm-5-minimax-25-chinese-la|latest open artifacts (#19): qwen 3.5, glm 5, minimax 2.5 — ]]
- [[entities/perplexity-internal-skill-design-guide-xiaojianke|perplexity 首次公开了内部 skill 设计指南]]
→ [[raw/articles/harness-之后-状态边界与失败闭环-若飞|原文存档]]^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]

- [[moc/data-infrastructure|MOC]]
## 深度分析

Harness 之后：Agent 可靠性的关键，是状态边界和失败闭环 涉及agent领域的核心技术议题。 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]
### 核心观点
1. # Harness 之后：Agent 可靠性的关键，是状态边界和失败闭环 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]
## 太长不看
- Harness Engineering 这轮讨论的价值，是把模型外面的执行环境、工具、上下文、生命周期、可观测、验证和治理，明确看成一个独立系统层（ETCLOVG 七层：Execution / Tooling / Context / Lifecycle / Observability / Verification / Governance）。
2. - 但 Harness 不能只写成组件清单。 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]
3. Agent 真进入工程流程以后，可靠性取决于这些组件能不能形成一套**状态清楚、证据可查、失败可恢复**的运行时闭环。 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]
4. - 长上下文不等于长期状态管理，memory 也不等于治理。 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]
5. 很多失败不是模型不会想，而是系统没有区分候选动作、已验证动作和已提交状态。 ^[raw/articles/harness-之后-状态边界与失败闭环-若飞.md]

### 关联实体

- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering]]

