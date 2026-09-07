---
title: "funes：Hugging Face 持久化 Agent 记忆层（traces 变可用记忆）"
created: 2026-09-05
updated: 2026-09-07
type: entity
tags: [agent-memory, funes, hugging-face, coding-agent, lance, retrieval, bm25, cross-encoder, vector-search, session, provenance, context-engineering]
sources: [raw/articles/give-your-coding-agents-a-memory-you-own]
confidence: 0.72
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# funes：Hugging Face 持久化 Agent 记忆层（traces 变可用记忆）

> Hugging Face 2026-09 发布 **funes**——给 coding agents 的持久化记忆层。主张"trace 是潜在记忆但只有索引、检索、排序、精确溯源后才是真记忆"。本地 Lance dataset + BM25×向量检索 + cross-encoder 重排 + recency 加权，`recall` 返回原文并精确溯源，可跨 agent / 跨机器 / 跨团队共享成私有 HF dataset。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

## 定位：记忆是 dataset，不是 service

funes 从机器上已有的 agent session 构建记忆，一条命令接入 Claude Code/Codex/pi/Hermes。核心设计立场是**"agent as a stranger" 问题**：每次会话结束，推理消失，每个新 agent 在新 host 上从零开始。funes 让 `recall` 变成 agent 日常工作流的一部分——触及过往决策、理由、发现时，agent 自行调用 `recall`，无需手动搬运旧 session 上下文。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

把记忆定义为 **dataset 而非 service**：本地是 Lance 的 append-only dataset（廉价增量写），共享是用户自有、默认私有的 HF dataset。凭据在索引时 redact，发布前二次扫描；agent 读远程记忆时本地缓存，warm 查询回到本地速度。Hub 只提供 ownership/ACL/versioning/distribution——不像独立记忆服务那样租回 API。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

## 技术架构

1. **统一 trace 形状**：确定性 pipeline 把各 agent 的 trace 解析成统一的 turn-and-block 形状，切块 → pinned 本地模型 embedding → 写本地 Lance dataset。索引增量：新 run 只加新 turn，旧内容有界回填。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]
2. **混合检索**：向量 + BM25 搜索 → 融合排序 → cross-encoder 重排 → recency 重加权 → 附加相邻 chunk。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]
3. **原始证据保留**：写入时不蒸馏成事实；`recall` 返回原文 + 精确来源（agent/时间戳/session/turn），`get` 打开完整上下文。这是"可搜索的 `CLAUDE.md`，保留为何如此的历史"。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

## `ask` 与 cross-agent 切换

`funes ask` 是只读、一问一答的 sibling：检索 passage → 交给 coding agent → 返回 grounded answer 并命名来源；检索 miss 不遮掩，直接说明。共享记忆不绑定创建 agent/model——Claude Code 开任务、Codex 下周续，第二个 agent 能 recall 第一个的推理。适用跨机器、跨团队（新成员第一天检索数月决策）、开源发布（release 背后的 session）。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

## handoff-vs-recall 基准

长 session 三方案对比：compaction（默认）、手写 handoff、recall。compaction 是唯一结果分化的（一任务到达一任务永不，失败处 fallback 摘要压平关键发现）；recall 返回 passage 本身，发现不靠幸存于摘要化。Recall 两任务都是最省：比 handoff 便宜 8x / 4x。^[raw/articles/give-your-coding-agents-a-memory-you-own.md]

## 相关实体

- [[entities/agent-memory-architecture-essence|Agent Memory 架构本质]]
- [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|Agent 记忆存储：wiki 编译 vs 原始数据之争]]
- [[entities/hermes-agent-memory-system-architecture|Hermes Agent Memory 系统架构]]
- [[entities/claude-code-agent-memory-four-levels-analysis|Claude Code Agent 记忆四层分析]]
- [[entities/ai-agent-memory-systems|AI Agent Memory 系统]]
- [[concepts/agent-memory-architecture|Agent Memory 架构]]

→ [[raw/articles/give-your-coding-agents-a-memory-you-own|原文存档]]