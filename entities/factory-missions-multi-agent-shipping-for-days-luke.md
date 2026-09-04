---

title: "一个 Mission 跑 16 天、烧 7.78 亿 Token：Factory 公开了多 Agent 系统的构建哲学"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, architecture, code, data, evaluation, llm, memory, mlops, multi-agent, observability, open-source, search, tool-use, trading, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/factory-missions-multi-agent-shipping-for-days-luke
---

# 一个 Mission 跑 16 天、烧 7.78 亿 Token：Factory 公开了多 Agent 系统的构建哲学

→ [[raw/articles/factory-missions-multi-agent-shipping-for-days-luke|原文存档]] ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]

## 深度分析

一个 Mission 跑 16 天、烧 7.78 亿 Token：Factory 公开了多 Agent 系统的构建哲学 ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
### 核心观点
1. # 一个 Mission 跑 16 天、烧 7. ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
2. 78 亿 Token：Factory 公开了多 Agent 系统的构建哲学 ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
> 整理自：Luke Alvoeiro @ AI Engineer Europe 2026-05
> 原文：Multi-Agent Systems / Missions That Ship for Days
> Factory 官方：https://factory.
3. ai/news/missions-architecture ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
## TL;DR
Factory 核心 agent 基础设施负责人 Luke Alvoeiro 的核心论点：**人类的注意力带宽已经成为软件工程的瓶颈**——前沿模型已经能并行处理 50 个任务，但即便最强的工程师同时也只能盯住 3-4 个 thread。 ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
4. Missions 是 Factory 针对这一不对称设计的多 agent 系统，目标是把工程师从「写代码」彻底搬到「项目管理 50 个 droid」。 ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
5. **值得抄作业的技术设计**： ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]
- 多 agent 通信归纳为 5 种基本模式，只用 4 种（不用 direct communication）
- Orchestrator + Worker + Validator 三角色
- Validation contract 在写代码之前产出
- 串行写、并行读
- Droid whispering：不同角色用不同 LLM
**真实数据**：Slack 克隆 mission，16. ^[raw/articles/factory-missions-multi-agent-shipping-for-days-luke.md]

### 关联实体

- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/构建基于多智能体架构的深度思考交易系统-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]

## 相关实体

- [[moc/multi-agent-coordination|MOC]]
