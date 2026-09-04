---

title: "从手动到自动化：用AgentEval构建Agent评测体系"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, architecture, code, data, database, evaluation, fine-tuning, llm, memory, observability, open-source, prompt, rl, security, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation
---

# 从手动到自动化：用AgentEval构建Agent评测体系

→ [[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation|原文存档]] ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]

## 深度分析

从手动到自动化：用AgentEval构建Agent评测体系 ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
### 核心观点
1. # 从手动到自动化：用AgentEval构建Agent评测体系 ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
> 来源：WalleZhang，2026-03-21
> GitHub：https://github.
2. com/wallezhang/agent-eval | 官网：https://agent-eval. ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
3. space ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
## 核心问题
Claude/Agent 评测的核心痛点： ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
- **非确定性**：同一 prompt 跑两次结果可能不同
- **传播效应**：改一个词可能导致整个行为链路变化，且不可预测
- **上游波动**：模型本身升级，Agent 表现可能波动
传统测试金字塔（单元→集成→E2E）覆盖不了 Agent 的核心质量问题。 ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
4. > "Teams without evals get bogged down in reactive loops — fixing one failure, creating another. ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
5. " — Anthropic ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]
## pass@k 和 pass^k 指标
- **pass@k**：k 次里至少有一次通过的概率（能力上限）
- **pass^k**：k 次全部通过的概率（可靠性）
两者一起看才有意义。 ^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]

### 关联实体

- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]

## 相关实体

- [[moc/evaluation-and-benchmarks|MOC]]
