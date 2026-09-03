---
title: "Working Set vs Long-Term Memory：Agent 上下文架构权衡"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, working-set, long-term-memory, context, memory, architecture]
sources: [entities/agent-harness-context-management-working-set, concepts/working-set-vs-long-term-memory, entities/存之有序治之有矩agent-记忆系统的工程实践与演进]
---

## 对照表

| 维度 | Working Set | Long-Term Memory |
|------|---|---|
| 存储位置 | LLM 上下文窗口（按 turn 计费） | 外部存储（file/DB/vector DB） |
| 容量 | 4K-200K tokens（模型上限） | 无限（硬盘的函数） |
| 读取延迟 | 零（已在上下文） | 高（I/O + search + embed） |
| 写入规则 | 每 turn 自动装填/discard | 显式 write / promote / consolidate |
| 刷新策略 | FIFO / 优先级裁剪 | access-count decay / TTL / 标记保留 |
| 召回方式 | 自动（模型 attention） | 按需（search / retriever / grep） |
| 管理工具 | harness context manager | memory system / wiki DB |

## 判断

好的 agent 系统两者都重——working set 是「快但贵」的工作内存，long-term 是「慢但便宜」的归档。设计目标：让 working set 装最需要的，让 long-term 能在 200ms 内召回。失败模式：working set 装太多 → token 爆炸；long-term 召回不准 → 模型「失忆」。

## 对比方来源

- [[concepts/working-set-vs-long-term-memory|working set vs long-term concept]]
- [[concepts/harness-context-window-management|harness 上下文窗口管理]]
- [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]]
- [[concepts/context-engineering|context engineering]]

## 进一步阅读

- [[entities/agent-harness-context-management-working-set]]
- [[concepts/working-set-vs-long-term-memory]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
