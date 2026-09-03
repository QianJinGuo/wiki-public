---
title: "上下文窗口经济学"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [context-management, cost, llm, token-economics]
provenance_state: inferred
confidence: 0.75
---

# 上下文窗口经济学

> 上下文窗口是 LLM 推理中最昂贵的资源——不是显存，不是算力，而是上下文中的每个 token 都在消耗金钱和注意力。

## 为什么上下文是经济学问题

LLM 推理成本与上下文长度成超线性关系（注意力计算 O(n²)）。Gemini 的 2M 窗口、Claude 的 200K 窗口不只是技术成就，更是经济主张。

## 成本结构分析

| 策略 | Token 消耗 | 质量影响 | 适用场景 |
|------|-----------|---------|---------|
| 全量加载 | 1× | 基线 | 短对话 |
| 分层记忆 | 0.3–0.5× | 微降 | 长会话 |
| 主动压缩 | 0.2–0.3× | 可控 | 极长任务 |
| RAG 按需 | 0.1–0.2× | 场景相关 | 知识查询 |

## 上下文压缩技术

1. **Taco 式丢弃**：识别并丢弃低价值上下文片段
2. **工作集压缩**：仅保留当前任务相关的"热"上下文
3. **摘要分层**：粗→细粒度渐进式摘要
4. **KV Cache 复用**：跨请求共享计算，避免重复编码

## Token 经济学五层模型

习惯优化 → 模型路由 → Context 压缩 → 代码图谱 → 多 Agent 协作

每往上一层，Token 节省的杠杆效应放大一个数量级。但组织复杂度同步上升。

## 关联

- [[concepts/context-management-framework|上下文管理框架]]
- [[entities/token-cost-control-coding-agent-devinyzeng-tencent|AI Coding Token 成本控制]]
- [[entities/taco-让-cli-agent-在自主迭代中学会丢掉无用上下文|Taco 上下文压缩]]
- [[concepts/inference-optimization|推理优化]]

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
