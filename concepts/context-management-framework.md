---
title: "上下文管理框架"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [context-management, llm, agent, framework]
provenance_state: inferred
confidence: 0.7
---

# 上下文管理框架

上下文管理是 Agent Harness 的第一支柱。LLM 的上下文窗口是有限的计算资源，需要像管理内存一样精细管理。

## 核心挑战

- **注意力塌缩**：长上下文中，中间位置信息利用率显著下降（Lost in the Middle）
- **上下文污染**：无关信息稀释有效信号，导致输出质量下降
- **成本爆炸**：Token 数量直接决定推理成本

## 工程对策

1. **分层记忆**：工作集（热）→ 短期记忆（温）→ 长期记忆（冷）
2. **主动压缩**：Taco 式上下文丢弃，保留高价值片段
3. **检索增强**：RAG 按需注入，避免预加载
4. **会话隔离**：SubAgent 分治，避免跨任务污染

## 关联

- [[entities/attention-collapse-context-management|注意力塌缩与上下文管理]]
- [[entities/taco-让-cli-agent-在自主迭代中学会丢掉无用上下文|Taco 上下文压缩]]
- [[entities/llm-inference-pipeline-internals|LLM 推理流水线]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
