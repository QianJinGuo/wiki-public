---
title: "Agent 评估框架/平台对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, agent-eval, benchmark, framework, vendor]
sources: [entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr, entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit, concepts/agent-evaluation-benchmarks, concepts/evaluation-harness-design]
---

## 对照表

| 维度 | AgentOps | lm-eval-harness | Agent-EvalKit | LangSmith |
|------|---|---|---|---|
| 生态 | AWS Bedrock | EleutherAI 学术 | AWS 开源 | LangChain |
| 评估对象 | agent 整体 | 模型/单 turn | agent CLI 行为 | agent trace |
| 评估方式 | LLM-as-judge + 第三方 | 标准 benchmark (MMLU/GSM8K) | AI coder + 6 阶段管线 | trace + human loop |
| 可观测 | ✅ 全栈 | ❌ 仅评分 | 中（流水线层面） | ✅ 全栈 |
| 成熟度 | 新（2026） | 成熟（2 年+） | 新（2026） | 中（1 年+） |
| 适合谁 | 企业 agent 上线前 | ML 研究/模型 benchmark | agent 开发者快速 eval | agent app 持续运行 |

## 判断

组合策略：模型选型用 lm-eval-harness 看 baseline；agent 开发期用 Agent-EvalKit 快速 iterate；上线后用 LangSmith + AgentOps 长期监控。单一工具都不够——评估三层（模型/agent/系统）各有 best practice。

## 对比方来源

- agent evaluation benchmarks
- [[concepts/evaluation-harness-design|evaluation harness 设计]]
- agent observability
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|Agent-EvalKit]]

## 进一步阅读

- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr]]
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit]]
- "Agent 评估基准体系"
- [[concepts/evaluation-harness-design]]
