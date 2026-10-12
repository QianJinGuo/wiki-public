---
title: "OpenRSI Index：科研智能体真实训练改进基准"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [rsi, benchmark, research-agent, training-infra]
sources: [raw/articles/从刷榜到做科研研究员还有多久被-ai超越]
confidence: 0.75
---

# OpenRSI Index：科研智能体真实训练改进基准

## 深度分析

OpenRSI Index 从真实开源研究项目出题，把已有模型/代码/数据/训练方案交给科研智能体在其上改进——反馈信号瓶颈的正面回应：当模型试图改进模型（RSI），谁来定义真实有效的环境反馈？三个代表任务：改进 Marin 优化器（六档模型 5.5 亿-25.45 亿参数，18432 H100·小时/次）、为 Qwen-122B 生成可验证 RL 任务（20864 H100·小时）、GPIC 视觉生成改进（5815 H100·小时，每候选对同千万张图最多再训一遍）。与刷榜评测的本质区别：任务要求真正运行训练实验，算力以千-万 H100·小时计。 ^[raw/articles/从刷榜到做科研研究员还有多久被-ai超越.md]

## 关联

- [[concepts/agent-self-improvement-loops|Agent 自改进循环]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]

→ [[raw/articles/从刷榜到做科研研究员还有多久被-ai超越|原文存档]]
