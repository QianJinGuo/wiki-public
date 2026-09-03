---
title: "情节记忆 vs 语义记忆：Agent 记忆系统设计对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, episodic, semantic, memory, agent, cognitive-architecture]
sources: [concepts/episodic-vs-semantic-memory-agent, entities/agent-memory-architecture, entities/存之有序治之有矩agent-记忆系统的工程实践与演进]
---

## 对照表

| 维度 | Episodic Memory | Semantic Memory |
|------|---|---|
| 定义 | 「用户上周说了什么」 | 「Python 列表语法是什么」 |
| 数据结构 | 时序索引（session_id + timestamp） | 主题索引（向量/图谱/SQL） |
| 读取模式 | 按时间/会话检索 | 按概念/关键词检索 |
| 写入模式 | session 结束自动 dump | consolidation 后写入 |
| 淘汰策略 | FIFO / TTL | access-count decay / 标记保留 |
| 幻觉风险 | 低（事实在原始会话中） | 高（摘要/归纳易失真） |
| Hermes 实现 | session DB（per session_id） | Memory / Wiki / Skills 三层 |

## 判断

两类不能混存——episodic 用时序索引（用户问「上次怎么说」时关键），semantic 用主题索引（用户问「这是什么」时关键）。混淆的代价：模型把对话当事实回答，产生「自指幻觉」。Hermes Memory 是 semantic（事实），session_search 是 episodic（对话），两者分库设计。

## 对比方来源

- [[concepts/episodic-vs-semantic-memory-agent|episodic vs semantic concept]]
- [[concepts/agent-memory-architecture|memory architecture]]
- [[concepts/memory-consolidation-decay|memory consolidation]]
- 认知架构 AI

## 进一步阅读

- [[concepts/episodic-vs-semantic-memory-agent]]
- [[entities/agent-memory-architecture]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
