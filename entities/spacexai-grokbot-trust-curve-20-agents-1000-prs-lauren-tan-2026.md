---

title: "SpaceXAI GrokBot：从 1 个 Agent 到 20 个并行的信任曲线"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai-coding, grokbot, spacexai, parallel-agents, trust-building, engineering-management, cursor, agent-orchestration]
sources: [raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026]
confidence: 0.7
provenance_state: extracted
---

## 概述

前 Cursor 工程师 Lauren Tan 在 SpaceXAI 负责 GrokBot 团队，分享了她如何从"盯着一个智能体不敢眨眼"进化到"放手让 20 个智能体自己合并主干"的五个月心路。核心洞察：**管智能体的技巧和管人的技巧，重合度高得惊人**。 ^[raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026.md]

## 核心框架：信任曲线

### 五个阶段

Lauren 分享了一张"信任曲线"：纵轴是信任，横轴是能同时开的智能体数量（从 1 到成千上万）。 ^[raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026.md]

1. **盯着一个智能体** —— 时刻盯着，每一行输出都要看，一句一句提示
2. **学会验证** —— 不是提示词技巧，而是验证能力
3. **并行 2-3 个** —— 开始信任智能体的基础能力
4. **并行 10-20 个** —— 完整任务交给智能体，自己只做监督
5. **成千上万** —— 系统化管理，人类只做高层决策 ^[raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026.md]

### 关键洞察

- **智能体在猜，而且它不知道自己在猜** —— 这是信任崩塌的根源
- **验证 > 提示** —— 重要的技能不是提示词，而是验证
- **管人技巧迁移** —— Netflix 工程经理经验直接适用于管智能体

### 实践数据

- 五个月，3000+ 个 PR
- 上个月交付 1000+ 个 PR
- 8 月目标：翻倍（2000+ PR）
- 同时跑 20+ 个 GrokBot，装在开源的 pstack 里
- 配合 /loop、/goal 和 /swarm，让智能体完整拥有任务

### 工具链

- **pstack** —— Lauren 开源的智能体管理框架
- **/loop** —— 循环执行任务
- **/goal** —— 目标驱动
- **/swarm** —— 多智能体协作

→ [[raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026|原文存档]] ^[raw/articles/spacexai-grokbot-trust-curve-20-agents-1000-prs-lauren-tan-2026.md]