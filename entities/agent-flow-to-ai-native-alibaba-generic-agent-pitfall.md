---
title: "从 Agent Flow 到 AI Native：通用 Agent 的架构反思"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [agent, architecture, flow, ai-native, skill, orchestration]
sources: [raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall]
confidence: 0.75
---

# 从 Agent Flow 到 AI Native：通用 Agent 的架构反思

阿里技术团队陈以强的一线研发视角，反思通用 Agent 的局限性，提出从 Agent Flow 到 AI Native 的演进路径。

## Agent Flow 设计

Agent Flow 是传统 Flow 的强化版：
- 用户用自然语言描述目标，LLM 理解目标、生成/修改/运行 Flow
- Flow 只是编排层，负责链路控制、任务下发和结果回收
- Node 可以是 QwenWork、Codex CLI、影刀 RPA、千牛接口、运维 Agent 等

## 核心案例

电商用户汇总报表：QwenWork 拿数据 → 影刀 RPA 拿数据 → 千牛接口拿数据 → 汇总 → DWS 写到钉钉文档 → 发到钉钉群。传统 Skill 无法在单个任务 loop 完成这个长链路，但 Agent Flow 可以无介入地跑通。

## 关键洞察

- Flow 本身的重要性降级——它只是一个编排层
- 核心理念：把任务切分下发，让节点负责执行
- 通用 Agent 的"通用"是双刃剑——在特定场景下，专用 Flow 比通用 Agent 更高效

## 与 Harness Engineering 的关系

Agent Flow 可视为 Harness 的一种具体实现：编排层 = Harness 框架，Node = 工具/技能，自然语言目标 = 任务定义。

→ [[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall|原文存档]]
