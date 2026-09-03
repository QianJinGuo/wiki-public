---
title: "Graph Engineering 来了：Claude Code 让 Agent 从一条直线变成一张图"
created: 2026-07-22
updated: 2026-08-29
type: entity
tags: ['graph-engineering', 'claude-code', 'dynamic-workflows', 'agent-orchestration', 'parallel']
sources: [raw/articles/graph-engineering-claude-code-seebin-2026]
provenance_state: extracted
---

> -> [[raw/articles/graph-engineering-claude-code-seebin-2026.md|原文存档]]

- **节点**：一个工作单元（Agent / 明确任务），输入→输出 ^[raw/articles/graph-engineering-claude-code-seebin-2026.md]

## 摘要

Seebin 基于 @0xCodez 的 "Graph Engineering with Claude" 长文整理出 14 种可复用的 Agent 图架构模式。核心概念只有两个：节点是一个工作单元（输入→输出），边是依赖关系；关键判断标准是每一条"然后"都要问——下一步读不读上一步的输出？不读就没有边，直接并行。文章提出 agent 编排的三次范式演进：Prompt Engineering（句子）→ Loop Engineering（循环）→ Graph Engineering（图），即让 Agent 从一条直线变成一张图 ^[raw/articles/graph-engineering-claude-code-seebin-2026.md]

14 种模式中最具代表性的包括：契约驱动节点（用 JSON schema 强制 subagent 返回结构化数据，输入输出有界只做一件事）、边即数据契约（用数据形状而非执行顺序命名边）、菱形拓扑（fan-out → reduce → synthesize，扇出收广度、纯代码压缩信息密度）、对抗性验证（验证节点坐在边上试图推翻发现，含 N 个怀疑者反驳、多视角验证、裁判团嫁接三种变体）、收敛循环（持续 spawn 查找器直到连续 K 轮无新发现，关键是去重所有已见结果）、模型分层（便宜模型做提取分类、最强模型做报告裁决）、Pipeline vs Barrier（parallel() 是 barrier 由最慢者决定总耗时，默认用流式 pipeline）。Claude Code 的 Dynamic Workflows 让 Claude 自己写编排脚本，三种入口：prompt 说 workflow、跑 /workflows、ultracode ^[raw/articles/graph-engineering-claude-code-seebin-2026.md]

## 关键要点

- 图架构的判定规则：每条"然后"检查下游是否消费上一步输出——不读就没有边，直接并行。
- 菱形拓扑（diamond）是标准形态：fan-out 收集广度 → 纯代码 reduce 压缩信息密度 → 最后一个 Agent synthesize 写出答案。
- 对抗性验证三模式：N 个独立怀疑者反驳（多数存活才保留）、正确性/安全性/可复现性多视角镜头、N 方案并行打分的裁判团（从赢家嫁接亚军精华）。
- 故障隔离：parallel() 中抛异常的 thunk resolve 成 null，.filter(Boolean) 即隔离墙；多 Agent 并行写文件冲突用各自 git worktree 隔离。
- 六个示例图：安全扫描、研究报告、逐文件移植、对抗性 diff 审查、定时生态扫描、未知规模发现。

## 来源

- 原文: [[raw/articles/graph-engineering-claude-code-seebin-2026.md|Graph Engineering 来了：Claude Code 让 Agent 从一条直线变成一张图]]
- 原始链接: : "https://mp.weixin.qq.com/s/x2oxCumvbeEaOXqHqmIK5w
