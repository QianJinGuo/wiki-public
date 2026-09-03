---
title: "Prompt Engineering vs Context Engineering"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, prompt-engineering, context-engineering, harness, agent]
sources: [concepts/prompt-engineering-patterns, concepts/context-engineering, entities/agent-harness-context-management-working-set]
---

## 对照表

| 维度 | Prompt Engineering | Context Engineering |
|------|---|---|
| 关注对象 | system prompt + user prompt 文本 | working set 组装 + 裁剪 + swap |
| 调优目标 | 让模型更懂任务/输出符合格式 | 让模型看到对的上下文/不噪音 |
| 方法论 | 模版、温度、few-shot、CoT | working set 优先级、滑动窗口、memory swap |
| 常用工具 | prompt 模版系统、A/B 测试 | harness context manager、memory 系统 |
| 谁负责 | 产品/prompt 工程师 | harness 架构师 |
| ROI 随模型提升 | 递减（强模型不需要 prompt 花样） | 递增（模型越强、欠/过上下文影响越大） |
| 成熟度 | 成熟（大量模式和最佳实践） | 仍在探索（标准未固化） |

## 判断

Prompt 工程是 Software 2.0 时代的元技能，context 工程是 Software 3.0 时代的元技能。强模型时代，prompt 调优的边际收益快速递减，但 context 工程的边际收益不断增加——这是 Karpathy 把 agentic engineering 核心定义为 harness/context/verifier 三件套的原因。

## 对比方来源

- [[concepts/prompt-engineering-fundamentals|prompt engineering 基础]]
- [[concepts/context-engineering|context engineering]]
- [[concepts/harness-context-window-management|harness 上下文窗口]]
- [[concepts/agent-engineering-capability-map|agent engineering 能力地图]]

## 进一步阅读

- "Prompt 工程模式"
- [[concepts/context-engineering]]
- [[entities/agent-harness-context-management-working-set]]
