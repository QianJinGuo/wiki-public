---
title: "RAG vs Agent Memory vs Context Window：何时用哪个？如何组合？"
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [rag, memory, context-engineering, architecture]
---

# RAG vs Agent Memory vs Context Window：何时用哪个？如何组合？

## 决策问题

> **在构建 Agent 系统时，三种信息保持机制——Context Window、显式检索（RAG）、Agent Memory——各自的职责边界是什么？什么时候该用哪个？它们如何组合才能避免"存储堆积却依然决策失效"？**

这个问题是所有 Agent 架构设计的核心取舍点。Context Window 扩展解决的是带宽问题，不是建模问题；RAG 本质是数据库思维的检索，而 Agent Memory 需要的是操作系统思维的治理闭环。三者定位不同，组合逻辑也不同。

---

## 三者本质对比

| 维度 | Context Window | RAG（显式检索） | Agent Memory |
|------|---------------|----------------|--------------|
| **本质** | LLM 的工作带宽，stateless 的临时工作区 | 外部知识库的精确召回 | 跨时决策影响力分配机制 |
| **类比** | RAM（工作内存） | 图书馆目录检索 | 对你的理解——跨期行为模式 |
| **解决的核心问题** | "现在上下文够不够" | "事实知识在不在" | "谁被允许持续影响未来决策" |
| **典型问题** | token 溢出、上下文耗尽 | 召回率、检索精度 | 连续性、遗忘、信念漂移、治理 |
| **信息保真度** | 最高（原始 token） | 中等（embedding + 截断有损） | 可变（Raw → Derived 逐级漂移） |
| **延迟** | 最低（无额外检索） | 中等（Reranker 是主要瓶颈 ~250ms） | 取决于架构复杂度 |
| **存储成本** | 最高（随 token 计费） | 低（外挂向量库） | 中等（三层热/温/冷架构） |
| **生命周期** | Session 结束即销毁 | 持久，但静态不动 | 持久，且需要主动管理更新/失效 |
| **数据模型** | 原始 token 序列 | 非结构化文档块 + embedding | 六维度结构：内容/类型/置信度/来源/作用域/时间衰减 |

^[raw/articles/how-ai-agent-memory-works.md]^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]^[raw/articles/agent-memory-architecture-essence.md]

---

## 决策树：何时用哪个

### 第一问：是否跨 Session 需要连续性？

```
当前问题
│
├─ 单 Session 内完成 ✓ → Context Window（直接放在 prompt 里）
│
└─ 需要跨 Session 记住 ✓
    │
    ├─ 是"我知道的事实"查找 ✓ → RAG（外部知识库检索）
    │
    └─ 是"我们对这件事的理解/偏好/结论" ✓ → Agent Memory
```

**Context Window 的触发信号：**
- 当前 Session 内的中间状态（工具调用结果、规划步骤）
- 用户当前正在讨论的话题的即时上下文
- 不需要跨 Session 保留的临时推理链
- 延迟敏感：任何外部检索都会影响响应时间

> 永远不要跳过"是否需要检索"的判断。对"翻译 hello 到日语"这类请求，调用 vector search 是纯粹的浪费。^[raw/articles/how-ai-agent-memory-works.md]

**RAG 的触发信号：**
- 事实性知识查询：文档、产品手册、代码库
- 需要精确回溯的场景：某行代码怎么写的、某个概念的定义
- 大规模语料库（100M+ tokens）：需要扩展性优先
- 知识库是共享的，不属于特定用户的个人理解

**Agent Memory 的触发信号：**
- 用户偏好、决策模式、沟通习惯
- 任务进度：上次做到了哪一步、哪些方案被否决了
- Agent 自我认知：哪个工具在什么场景下失败过
- 需要置信度追踪：新信息如何修正旧信念
- 需要遗忘机制：什么该退出舞台

^[raw/articles/agent-memory-architecture-essence.md]^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]

---

## 何时用 Context Window

Context Window 是 LLM 的"RAM"——掉电即失，但速度最快、成本最低（不考虑 token 费用时）。

**具体使用场景：**

1. **当前 Session 的对话历史**：用户和 Agent 正在进行的对话，全部放在 context 里
2. **工具调用的中间结果**：API 返回值、数据库查询结果，用于下一步推理
3. **规划器的当前步骤**：Agent 正在执行的多步计划，不需要持久化
4. **系统提示词前缀**：MEMORY.md 和 USER.md 的快照，在会话期间作为 frozen snapshot 注入

> Hermes Agent 的热记忆刻意限制：MEMORY.md + USER.md 的 2,200 + 1,375 字符限制，用字符而非 token 限制，解耦对特定模型 tokenizer 的依赖。这两块内容在会话期间作为 frozen snapshot 注入系统提示词，写入时落盘但不修改已构建的 prompt，保护 prompt cache 的命中稳定性。^[raw/articles/how-ai-agent-memory-works.md]

**Context Window 的局限：**
- Session 结束即销毁，无法跨会话
- 容量有上限（即使扩展到 1M token，存储成本也极高）
- 不是永久存储，是工作区

---

## 何时用 RAG

RAG = 外部知识库的精确检索。本质是"图书馆"——回答"这个知识在不在"，不回答"我们之前对这件事形成过什么共识"。

**RAG 最适合的场景：^[raw/articles/context-engineering-three-memory-paradigms-comparison.md]**

| 场景特征 | 推荐原因 |
|----------|----------|
| 需要精确回溯、细节还原 | 无损存储，检索即还原 |
| 延迟敏感、快速响应 | 延迟最低（~2249ms），架构简单 |
| 追求零 context 推理 | 可以完全不依赖上下文 |
| 小说情节、多轮对话上下文 | 精确上下文还原优于 MSA |
| 垂直领域（医疗、法律） | 需用 fine-tuned domain embedding |

**RAG 的设计要点：**
- HyDE + RRF 是 production retrieval 标配组合，可将召回率提升 20-30%^[raw/articles/how-ai-agent-memory-works.md]
- Embedding 模型一旦更换，所有历史向量都需要重新计算^[raw/articles/how-ai-agent-memory-works.md]
- Reranker 是最大瓶颈（250ms），许多生产系统选择轻量级 reranker 用少量相关性损失换取显著延迟改善^[raw/articles/how-ai-agent-memory-works.md]

**RAG 不适合的场景：**
- 需要捕捉用户偏好演变的场景
- 需要跨 Session 的决策连续性
- 需要置信度追踪和信念修正
- "我们上次讨论时否决了这个方案"这类历史上下文

---

## 何时用 Agent Memory

Agent Memory 不是"更聪明的检索"。把 Memory 误解为检索 → 架构做偏：以为问题在召回率，实际问题在连续性。^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]

**Memory 的核心职责是 write–manage–read 闭环，而非存储：^[raw/articles/agent-memory-architecture-essence.md]**

- **写入**：不是"多记一点"，而是"尽量别记错"——边际价值优先于绝对价值^[raw/articles/how-ai-agent-memory-works.md]
- **管理**：整合、冲突调解、版本替换、衰减归档、标记失效^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]
- **读取**：任务约束驱动的检索-推断耦合，而非语义相似度召回^[raw/articles/agent-memory-architecture-essence.md]
- **遗忘**：不是粗暴擦除，而是谱系清算——追溯这条信息去过哪里、变成过什么、影响过哪些派生物^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]

**Agent Memory 最适合的场景：**

| 场景 | Memory 为什么必要 |
|------|------------------|
| 用户偏好的形成与修正 | 需要置信度、来源追踪、冲突保留 |
| 跨 Session 的任务连续性 | "上次做到了哪步，哪些方案被否决" |
| Agent 自我认知 | "这个工具在什么场景下失败过" |
| 多 Agent 共享记忆 | 权限拓扑、隔离、一致性、可追责 |
| 需要选择性遗忘 | 防止旧判断锁死现实 |

**Memory 的四种建模对象：^[raw/articles/agent-memory-architecture-essence.md]**

1. **用户模型**：偏好、风险偏好、沟通习惯、决策模式
2. **任务模型**：被否决的方案、已确认的结论、当前真版本、未完成的承诺
3. **世界模型**：仓库结构、API 约束、系统边界、组织规则
4. **自我模型**：试过什么、哪条路径失败、哪个工具不稳定

---

## 如何组合：互补使用模式

三种机制不是互斥的，而是分工明确的互补系统。

### 组合一：RAG（底层知识）+ Memory（个人理解）+ Context（当前工作区）

```
当前请求
    │
    ▼
Context Window ←── 当前 Session 中间状态
    │
    ├── Memory ──→ 用户偏好 / 任务历史 / Agent 自我认知
    │
    └── RAG ────→ 共享知识库 / 产品文档 / 代码库
```

**典型场景：企业销售 Agent**
- Context Window：当前对话轮次 + 正在查看的产品页
- Memory：用户的采购历史、决策人关系、上次否决的理由
- RAG：产品规格文档、公司最新政策、竞品对比表

### 组合二：MemGPT 式的分层架构

| 层级 | 存储介质 | 内容 | 成本 |
|------|----------|------|------|
| HOT（热） | Redis / 内存 KV | 活跃用户的核心偏好、当前任务状态 | 最高 |
| WARM（温） | Vector DB | 近期对话的 episodic 记忆 | 中等 |
| COLD（冷） | S3 / 对象存储 | 历史会话日志、归档 | 最低 |

三层架构可将热数据检索延迟降低一个数量级，存储成本降低 70-80%。^[raw/articles/how-ai-agent-memory-works.md]

### 组合三：Claude Code vs OpenClaw 的殊途同归

两种 Agent 记忆系统的对比揭示了架构收敛方向：^[raw/articles/claude-code-openclaw-memory-comparison.md]

- **语义初筛（向量）** → 快速召回候选
- **候选精选（LLM）** → 上下文感知筛选
- **精确读取（行号范围）** → 最小 context 消耗

> 最终的记忆系统大概率是混合架构：RAG 提供精确细节检索，MSA 提供语义层面的快速匹配，参数记忆提供任务特定的行为模式。^[raw/articles/context-engineering-three-memory-paradigms-comparison.md]

### 组合四：Pre-compaction Flush + Session Summary

OpenClaw 的 Pre-compaction Flush（在压缩前强制 Agent 将重要信息写入 MEMORY.md）是防止长会话信息丢失的关键设计。Claude Code 的 Session Memory 摘要则是压缩后保留下限的机制。两者结合：^[raw/articles/claude-code-openclaw-memory-comparison.md]

```
上下文窗口快满
    │
    ├── Pre-compaction Flush（压缩前）→ 强制写入 MEMORY.md
    │
    └── Session Memory 摘要（压缩后）→ 滚动摘要作为中层记忆
```

---

## 常见错误

### 错误一：把 Memory 当"更聪明的 RAG"

**症状**：只关注召回率优化，加入 embedding 模型和向量检索，却发现 Agent 依然"记了却决策失效"。

**原因**：Memory 的核心问题不是召回，是连续性——RAG 回答"我之前有没有谈过这个"，Memory 回答"之前形成的结论现在还成不成立、要不要更新"。^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]

**正确理解**：Memory 是 write–manage–read 闭环，RAG 只是读取路径的一个子模块。

---

### 错误二：只写入不管理——积累误解而非智慧

**症状**：系统记住越来越多信息，但 Agent 基于"记忆"做出的决策越来越差。

**原因**：没有管理链路，Raw 材料被反复蒸馏后系统性漂移：语气 → 语境 → 边界条件 → 例外项 → 时间限定。^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]

**正确理解**：
- 每次压缩都应尽可能回到证据层校验^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]
- 冲突信号（保守用户突然要求尝试新框架）= 高价值信号，值得优先写入^[raw/articles/how-ai-agent-memory-works.md]

---

### 错误三：Context Window 当长期存储用

**症状**：把大量历史对话塞进 Context Window，试图靠"足够长的上下文"替代 Memory。

**原因**：Context window 扩展解决的是带宽问题，不是建模问题。benchmark 已证实：拉到 35 session、300 turn 的尺度，长上下文和 RAG 在时间推理、长程一致性上仍明显落后于人类。^[raw/articles/agent-memory-architecture-essence.md]

**正确理解**：Context Window 是工作区（RAM），不是存储（磁盘）。

---

### 错误四：把摘要当记忆，丢失信念修正轨迹

**症状**：Agent 说"我记得用户偏好 TypeScript"，但无法回答"用户什么时候开始偏好 TypeScript 的、后来有没有改变过"。

**原因**：摘要能写下结论，留不下形成结论的轨迹。蒸馏擅长留下结论，不擅长留下形成结论的上下文。^[raw/articles/agent-memory-architecture-essence.md]

**正确理解**：记忆单元需要六个维度：内容/类型/置信度/来源/作用域/时间衰减。其中"来源"决定了这条记忆是"原始证据"还是"Agent 推断"。

---

### 错误五：跳过"是否需要检索"的判断

**症状**：对简单请求（如翻译、计算）调用 vector search，浪费延迟和成本。

**正确理解**：永远不要跳过"是否需要检索"的判断。如果 query 不需要任何记忆内容，就不要调用检索系统。^[raw/articles/how-ai-agent-memory-works.md]

---

### 错误六：多 Agent 记忆共享时忽略权限拓扑

**症状**：跨 Agent 记忆泄漏、某些 Agent 看到不该看到的信息。

**原因**：多 Agent 记忆的核心问题从"如何存储"变成**权限拓扑**问题——谁能看到什么。六个失败模式中最危险的是 Cross-user leakage（隐私泄露）。^[raw/articles/how-ai-agent-memory-works.md]

**正确理解**：多租户隔离是生死线，每个租户的记忆必须严格隔离。Namespace per tenant 隔离严格但成本高；single collection + payload filter 成本低但存在隔离失效风险。

---

## 关键判断指标速查

| 问题 | 答案方向 |
|------|----------|
| 需要跨越 300 个 Session 记住用户的偏好？ | → Memory |
| 需要在 100M tokens 的文档库里找一条具体定义？ | → RAG |
| 当前 Session 内的中间推理状态？ | → Context Window |
| 需要知道"用户上次为什么否决了这个方案"？ | → Memory |
| 需要查"这个 API 的具体参数说明"？ | → RAG |
| 用户的偏好是否在最近发生了漂移？ | → Memory（漂移检测） |
| 延迟 p95 要求 < 500ms？ | → Context Window 优先 |
| 需要支持多 Agent 共享同一记忆？ | → Memory + 权限治理 |
| 需要让用户能查看/编辑/删除记忆？ | → Memory（治理层） |

---

## 相关实体

-  — Mert Cobanov 的系统性梳理
-  — 生命周期管线、治理层、五种架构
-  — MSA / RAG / Doc-to-lora 实验数据
-  — write–manage–read 闭环、四建模对象、六维度
-  — LLM 语义路由 vs 向量+全文混合搜索
- [[entities/agent-memory-architecture|Agent Memory 架构本质]] — 治理 vs 存储的核心判断
-  — 热/温/冷存储设计
-  — 分层成本治理实践
- [[concepts/agent-memory-systematic-framework|Agent Memory 系统性框架]] — 核心概念链接

---

## 参考来源

- [[raw/articles/how-ai-agent-memory-works.md|how-ai-agent-memory-works — Mert Cobanov]]
- [[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md|memory-vs-rag-agent-memory-systematic-framework — 微信综述]]
- [[raw/articles/context-engineering-three-memory-paradigms-comparison.md|context-engineering-three-memory-paradigms-comparison]]
- [[raw/articles/agent-memory-architecture-essence.md|agent-memory-architecture-essence — lencx/浮之静]]
- [[raw/articles/claude-code-openclaw-memory-comparison.md|claude-code-openclaw-memory-comparison — 行小招/科技充电站]]
