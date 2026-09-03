---
source_url:
title: "Hermes Agent 三级 Memory 架构解析（One掌柜视角）"
created: 2026-05-19
updated: 2026-09-01
summary: Hermes Agent 三层 memory 架构：Layer 1 MEMORY.md+USER.md 常驻 system prompt（frozen mid-session）、Layer 2 SQLite+FTS5 历史按需检索、Layer 3 可插拔 semantic provider；配合 Periodic Nudge 每 300s 自主整理写回。
review_value: 7
sources: []
review_confidence: 8
review_recommendation: worth-reading
review_stars: 3
tags: [hermes-agent, memory, architecture, sqlite-fts5, semantic-memory]
related_entities:
 - entities/hermes-agent-memory-system-architecture
 - entities/我给hermes配了4个agent真正有用的是这些事
 - entities/深度拆解-hermes-agent-记忆系统它修正了-openclaw-的哪层误区
type: entity
provenance_state: inferred
---
## 与 VibeCoder 版本的关系

本文与 VibeCoder 的《Hermes Agent Memory System 架构解析》是**同一源码的独立分析**，但切入角度不同：

- VibeCoder 版本侧重框架源码结构和 MCP Tool 机制
- 本文（One掌柜）侧重架构分层设计和 memory 流转逻辑

VibeCoder 从代码层面拆解了 Hermes 的 memory provider 注册、FTS5 索引构建、以及 session_search 的 FTS5 语法细节；而本文更关心**为什么这么分层、每一层解决了什么工程问题、以及这套架构对 AI Agent 记忆系统设计的普遍启示**。两篇互为补充，建议对照阅读。

## 三层架构

| 层次 | 名称 | 载体 | 访问方式 | 特点 |
|------|------|------|---------|------|
| Layer 1 | Markdown Memory | MEMORY.md (~2200字) + USER.md (~1375字) | 每轮 system prompt | frozen mid-session，80% consolidation |
| Layer 2 | 历史检索层 | SQLite + FTS5 | session_search(query) | 10ms 检索万级 docs，Gemini Flash 总结 |
| Layer 3 | Semantic Provider | 外部可插拔 provider | PREFETCH/SYNC/EXTRACT | Opt-in，多 provider 支持 |
| 额外 | Periodic Nudge | — | 每 300s | autonomous curation，什么值得写回 memory |

这三层并非简单的堆叠，而是**刻意解耦的独立子系统**。Layer 1 和 Layer 2 完全运行在本地，不需要任何外部 API；Layer 3 则是一个可选的扩展层，支持接入向量数据库、语义搜索等外部能力。这种设计意味着：即使关闭所有外部依赖，Hermes 的核心记忆能力依然完整可用。

Layer 1 的 MEMORY.md 和 USER.md 是**有容量预算的精选摘要**——MEMORY.md 约 2200 字、USER.md 约 1375 字，加起来不超过 system prompt 的一个 token 片段。这个预算不是随意设定的，而是经过 80% consolidation 后的"精华浓缩"——大部分冗余信息已经被压缩掉，只保留对当前对话最有价值的记忆碎片。

## 关键设计洞察

**1. Mid-session Frozen 保护 Prefix Cache**

本轮 memory 变更不立即打乱 prefix cache，下轮才注入。说明 Hermes 在平衡：记忆写入 vs prompt 稳定性 vs prefix cache 成本。这不是一个简单的延迟写入策略，而是对 LLM 推理经济学的深刻理解——prefix cache 命中率直接影响每次推理的 token 成本，而 system prompt 的稳定性是 prefix cache 生效的前提。

**2. Tier 1+2 独立于 Tier 3**

就算切换 semantic provider，Layer 1 和 Layer 2 的能力不受影响，Tier 3 是可选项。这体现了"渐进增强"的设计哲学——核心功能不依赖外部服务，扩展能力按需开启。对于个人用户来说，Layer 1+2 已经能覆盖 90% 的记忆需求。

**3. Autonomous Curation：AI 自己决定什么值得记住**

不是等用户手动喂记忆，而是系统周期性判断"什么值得留下"——Periodic Nudge 是主动式 memory 管理机制。这打破了传统 RAG 系统"被动等待检索"的模式，让 AI 自己充当记忆的策展人。

**4. 成本分层：从零成本到可选付费**

三层架构天然对应三种成本模型：Layer 1 零成本（纯本地 Markdown），Layer 2 极低成本（SQLite 本地检索），Layer 3 按需付费（外部语义搜索 API）。这种设计让用户可以根据预算和需求自由选择，而不是被迫为所有功能付费。

**5. FTS5 替代向量搜索的务实选择**

Hermes 没有默认使用向量数据库做语义搜索，而是选择了 SQLite FTS5 这个轻量级全文检索引擎。10ms 检索万级文档的性能已经足够满足 Agent 的记忆检索需求，而且完全本地运行、零依赖。这是一个典型的"够用就好"的工程决策。

**6. Memory 是有生命周期的**

Periodic Nudge 不只是"整理记忆"，而是在做**记忆的生命周期管理**——判断什么值得写回 MEMORY.md（长期记忆）、什么留在 SQLite 供检索（中期记忆）、什么可以丢弃（过期信息）。这种主动遗忘机制比"无限存储"更重要。

## 相关框架对比

| 框架 | Memory 方案 | 特点 | 成本模型 | 本地能力 |
|------|-----------|------|---------|---------|
| Hermes | 三层（Markdown + SQLite FTS5 + Semantic Provider） | 可控、成本低、可插拔 | 零成本→按需付费 | Layer 1+2 完全本地 |
| OpenClaw | 单层 RAG | 外部检索为主 | 依赖外部 API | 需要向量数据库 |
| Cursor | Context 自动压缩 | 基于上下文窗口管理 | 包含在订阅中 | 本地执行 |
| Claude Code | 七层 Memory 架构 | 多层文件系统记忆 | 包含在订阅中 | CLAUDE.md 本地 |
| ChatGPT Dreaming | 长期记忆架构 | 自动提取+用户确认 | OpenAI 平台 | 需要云端 |

> 来源：[One掌柜](https://mp.weixin.qq.com/s/xWphR-dDs5c64FgEggRDmw)

从对比可以看出，Hermes 的独特之处在于：**它把记忆系统的控制权完全交给了用户**。MEMORY.md 和 USER.md 是纯文本文件，用户可以直接编辑、版本控制、备份；SQLite 数据库可以离线查询；即使接入外部 semantic provider，也是 opt-in 而非默认。这种"用户可控"的设计哲学，在当前 AI Agent 框架中并不多见。

## 深度分析

### Mid-session Frozen 的 Prefix Cache 经济学

Hermes 选择"本轮 memory 变更下轮才生效"，这个看似简单的延迟写入策略，背后是**prefix cache 经济学**的精确计算。

现代 LLM 推理服务（如 vLLM、TensorRT-LLM）普遍支持 prefix caching——当多个请求共享相同的 system prompt 前缀时，可以复用 KV cache，避免重复计算。对于 Hermes 来说，MEMORY.md 和 USER.md 构成 system prompt 的一部分，如果每轮都修改这两个文件，prefix cache 的命中率会急剧下降。

假设一轮对话中 memory 变更了 3 次，如果不 frozen，prefix cache 会失效 3 次，每次都需要重新计算整个 system prompt 的 KV cache。而 frozen mid-session 确保了**同一轮对话内 system prompt 稳定**，prefix cache 可以持续命中，推理成本大幅降低。

这是一种典型的"牺牲即时性换取效率"的工程妥协——用户在本轮对话中看到的记忆可能略微滞后，但整体系统的推理成本和响应速度都得到了优化。在实际使用中，这种延迟几乎不可感知，因为用户通常不会在单轮对话中频繁修改核心记忆。

### FTS5 vs 向量搜索：轻量级检索的务实哲学

Hermes 的 Layer 2 选择 SQLite FTS5 而非向量数据库，这个决策体现了**务实的工程判断**。

向量搜索（如 Pinecone、Weaviate、ChromaDB）在语义相似性检索上确实更强，但它们带来三个显著成本：外部依赖（需要运行额外服务）、延迟增加（网络请求 + 向量索引查询）、以及运维复杂度（向量数据库的备份、扩展、故障恢复）。

SQLite FTS5 在以下方面已经足够好：
- **性能**：10ms 检索万级文档，对于 Agent 的记忆检索场景绰绰有余
- **精度**：全文检索 + BM25 排序在精确关键词匹配上甚至优于向量搜索
- **可靠性**：本地文件，无网络依赖，零运维成本
- **容量**：单个 SQLite 文件可以存储数百万条记录

更重要的是，FTS5 支持复杂的布尔查询、通配符搜索、以及 BM25 排序——这些在实际的记忆检索中比纯语义相似性更实用。比如用户说"上次讨论的那个部署方案"，FTS5 可以精确匹配"部署"+"方案"这两个关键词，而向量搜索可能会返回语义相似但不相关的内容。

### Periodic Nudge 作为自主记忆策展

Periodic Nudge 每 300s 运行一次，不是简单的"整理记忆"，而是一个**自主策展系统**——AI 自己判断什么值得写回 MEMORY.md、什么留在 SQLite、什么可以丢弃。

这个设计借鉴了认知科学中的"间隔重复"和"选择性遗忘"理论。人类记忆并非无限存储，而是通过海马体的定期整合，将短期记忆转化为长期记忆，同时丢弃不重要的信息。Periodic Nudge 模拟了这个过程：

1. **评估重要性**：判断某条信息是否值得进入长期记忆（MEMORY.md）
2. **压缩存储**：将冗余信息压缩成精炼摘要
3. **主动遗忘**：丢弃过时或低价值的信息，防止记忆膨胀

与传统 RAG 系统的"被动等待检索"不同，Periodic Nudge 是**主动式记忆管理**——不是用户指挥 AI 记住什么，而是 AI 自己决定什么值得沉淀。这在 Hyperbolic Lab 等项目中也有类似实践，但 Hermes 将其做成了周期性后台任务而非实时拦截，避免了干扰正常对话流。

### 成本优先的分层设计哲学

Hermes 的三层架构本质上是一个**成本梯度模型**：

- **Layer 1（零成本）**：纯本地 Markdown，不需要任何 API 调用
- **Layer 2（极低成本）**：SQLite 本地检索，单次查询 < 0.001 美元
- **Layer 3（按需付费）**：外部语义搜索 API，按调用次数计费

这种设计让用户可以根据预算和需求自由选择，而不是被迫为所有功能付费。对于个人用户来说，Layer 1+2 已经能覆盖 90% 的记忆需求；对于企业用户或高频使用者，Layer 3 提供了更强的语义检索能力。

更重要的是，这种分层确保了**核心功能不依赖外部服务**。即使关闭 Layer 3、甚至断开网络，Hermes 的记忆系统依然完整可用。这对于注重隐私和数据主权的用户来说，是一个重要的设计优势。

### Memory 作为有生命周期的系统

Hermes 的记忆管理不是"存储越多越好"，而是**记忆有生命周期**——从短期（对话上下文）到中期（SQLite 检索）到长期（MEMORY.md），每个阶段都有明确的保留策略和淘汰机制。

这种设计避免了"记忆膨胀"问题。传统 RAG 系统往往只做"存入"不做"清理"，导致检索库越来越大、噪声越来越多、检索精度逐渐下降。Hermes 通过 Periodic Nudge 的主动遗忘机制，确保 MEMORY.md 始终保持精炼——大约 2200 字的容量预算本身就是一种强制约束。

记忆生命周期管理还包括**版本控制**：MEMORY.md 是纯文本文件，可以用 Git 追踪变更历史；SQLite 数据库可以定期备份；即使 memory 被错误修改，也可以回滚到之前的版本。这种"可恢复性"在长期使用中至关重要。

## 实践启示

1. **个人使用场景下，Layer 1+2 足够**：如果你用 Hermes 处理日常任务，Layer 1 的 MEMORY.md + USER.md 加上 Layer 2 的 SQLite FTS5 检索已经能覆盖 90% 的记忆需求。Layer 3 的 semantic provider 是锦上添花，不必强求。

2. **利用 Periodic Nudge 优化 Memory 质量**：不要让 memory 变成垃圾堆。每隔一段时间（比如每天）检查一次 Nudge 写回的内容，确保 MEMORY.md 保持精炼、不过时。Memory drift 是长期使用的大敌。

3. **分离"研究与执行"Agent 是高效策略**：vmiss 的四 Agent 配置中，研究 Agent 和执行 Agent 分离——研究 Agent 负责学习、总结，执行 Agent 负责技能构建。这种分工避免了"一个 Agent 什么都在做但什么都做不深"的问题。

4. **本地模型 + 云端模型混合使用可行**：vmiss 用 RTX 4070 8GB 跑 Qwen 3.5 9B 量化模型处理健康咨询类任务，效果"最让人惊讶"。这说明对于特定垂直场景，本地小模型已经足够好用，而且零 API 成本。

5. **记忆系统的控制权应该交给用户**：Hermes 让 MEMORY.md 和 USER.md 作为纯文本文件存在，用户可以直接编辑、版本控制、备份。这种"用户可控"的设计哲学比黑箱式的自动记忆管理更值得借鉴——至少你应该知道 AI 记住了什么、可以随时修改或删除。

6. **prefix cache 优化应该成为 memory 系统设计的必修课**：随着 LLM 推理成本成为核心考量，任何影响 system prompt 稳定性的 memory 操作都需要考虑 prefix cache 的影响。Hermes 的 mid-session frozen 策略提供了一个实用的参考模式。

## 相关实体

- [[queries/agent-memory-system-design|Agent Memory System 设计指南]]
- [[entities/how-ai-agent-memory-works|AI Agent 记忆系统架构]]
- [[entities/hermes-agent-memory-system-vs-openclaw|Hermes Agent 记忆系统深度拆解]]
- [[concepts/agent-memory-system-design|Agent Memory System Design]]
- [[entities/hermes-agent-three-layer-memory-architecture-one|Hermes Agent 三级 Memory 架构]]
- [[queries/hermes-agent-core-architecture-self-evolution|Hermes Agent 核心架构与自我进化]]
- [[entities/hermes-agent-memory-system-architecture|Hermes Agent 记忆系统（VibeCoder 版）]]
- [[entities/claude-code-7-layer-memory-architecture|Claude Code 七层 Memory 架构]]
- [[concepts/agent-memory-substrate-three-layer|Agent Memory 底层三层抽象]]
- [[concepts/context-window-economics|上下文窗口经济学]]
- [[entities/ai-memory-architecture-deep-dive|AI Memory 架构深度拆解]]
