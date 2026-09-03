---
title: Agent Memory 系统性框架
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [ai-memory, rag, architecture, agent]
related:
  - [[entities/agent-harness-architecture|Agent Harness 架构]]
  - [[entities/claude-code-source-architecture|Claude Code 源码拆解：从启动到多 Agent 扩展层]]
sources: ['raw/articles/memory-vs-rag-agent-memory-systematic-framework']
confidence: high
---
# Agent Memory: A Systematic Framework
> "Memory is not a smarter retrieval" — this piece establishes a comprehensive framework for Agent Memory as a lifecycle system: Write → Organize → Read, Raw vs. Derived materials with the "broken telephone" effect, the necessity of forgetting, Skills as procedural memory externalization, the MemGPT OS metaphor, five architecture philosophies, governance layer, multi-Agent sharing, and an evaluation framework.
---
## Core Thesis: RAG is not Agent Memory
| | RAG | Memory |
|--|-----|--------|
| Nature | "Reading" — knowledge coverage | Full lifecycle: write / update / replace / invalidate / delete |
| Analogy | Library — covers knowledge range | Understanding of you — handles individual relationships and behavioral evolution |
| Core question | Recall rate | Continuity; what to keep / update / invalidate |
**The fundamental error**: Treating Memory as "smarter retrieval." Architecture becomes misaligned:
- You think the problem is recall rate → actual problem is **continuity**
- You think historical search is sufficient → actual problem is **knowing what to preserve, update, invalidate**
---
## Memory Lifecycle Pipeline
Memory is never "stored once" — it is **continuously lossy reconstructed** through three stages.
### Stage 1: Write (Deciding What's Worth Remembering)
**Core difficulty**: Future utility is unknown at write time.
Three required actions:
1. **Extract candidate memories**: From conversations, tool calls, environmental observations → identify what might be worth long-term preservation
2. **Write gating**: Low-value noise shouldn't enter memory indiscriminately. Goal is "remember as accurately as possible," not "remember everything"
3. **Metadata annotation**: Type, source, timestamp, confidence. Without metadata, it's just text fragments.
> **The first failure mode is dirty writes, not wrong reads.**
### Stage 2: Organize (Letting Old Information Exit)
**Core operations**: Deduplication, conflict mediation, version replacement, decay archiving, invalidation marking.
**The real questions**:
- Does this memory still hold now?
- Does it conflict with later information?
- Should it be updated, replaced, or kept as historical evidence?
> **A system that only accumulates but never organizes is not accumulating wisdom — it's accumulating misunderstanding.**
### Stage 3: Read (Temporarily Assembling a Working Hypothesis of the Past for the Present)
Mature reading = hybrid retrieval + reranking + filtering + budget裁剪 + context assembly.
**Key insight**: The system doesn't "restore the past" — it "reconstructs an actionable past version for the current task."
> **Memory is not a factual repository, but a mechanism for continuously reconstructing past meaning.**
---
## Raw vs. Derived: The "Broken Telephone" Effect
### Two Types of Materials
| Type | Characteristics | Problem |
|------|----------------|---------|
| **Raw** | Complete conversation logs, tool call traces, environmental observations | Too scattered, fragmented, expensive; lacks actionable meaning |
| **Derived** | Summaries, user profiles, preference tags, relationship graphs | Already interpreted and compressed; drifts from facts progressively |
### The Drift Mechanism
> Content repeatedly rewritten, repeatedly summarized, repeatedly used to generate new summaries → systematic information drift.
**Loss order**: Tone → context → boundary conditions → exceptions → temporal constraints.
**Result**: The system may retain not the truth, but an increasingly smooth-sounding, increasingly unreliable version.
### Resolution Principles
- **Without evidence layer → drift**
- **Without derived layer →钝化 (become obtuse)**
- Every compression should trace back to evidence layer for verification
- High-quality architecture: Raw + Derived constrain each other
---
## Memory 与 Harness 的边界：谁该管什么

Agent Memory 系统不是孤立存在的——它需要在更广泛的 Harness 架构中找到自己的位置，理解 Memory 与其他 Harness 组件之间的边界和交互，是设计可维护系统的关键。

**Memory 与 State 的边界**。State 是当前 session 内的短期运行态，Session 结束即销毁；Memory 是跨 session 持续存在的历史对象。 State 提供当前上下文的实时快照（当前任务目标、工具调用进度、中间结果），Memory 提供跨时间的持续性（用户偏好变化、已失败的路径、长期积累的世界模型）。State 是写入 Memory 的原材料——每次 session 结束时，State 中有价值的部分被抽取、压缩、写入 Memory；Memory 是 State 的历史锚点——新 session 开始时，从 Memory 中检索相关内容来初始化 State。

**Memory 与 Policy 的边界**。Policy 定义权限边界、安全规则、合规约束，是外部强加的规范，Policy 层不应被 Memory 系统动态修改。Policy 决定什么可以做、什么不可以做；Memory 决定谁知道什么、什么应该影响当前决策。Memory 不能覆盖 Policy——一个用户可能"习惯性地试图访问某个受限资源"，但 Policy 不会因为 Memory 的这条记录而改变规则，但 Memory 可以让 Agent 在面对该用户时更早地触发确认流程。

**Memory 与工具系统的交互**。工具调用结果（API 响应、文件内容、环境状态变化）是 Memory 的重要输入来源——它们提供了关于外部世界的客观观测，而非用户的转述或 Agent 的推断。工具调用日志中的重复失败模式是自我模型（Self Model）的关键输入：某个工具在什么场景下不稳定、哪条路径已经被证明失败过，这些都应该被 Memory 系统记录并在工具选择时被查阅。

**Memory 与 Evaluator 的交互**。Memory 支撑 Evaluator 的验证能力——当 Evaluator 需要判断当前状态是否满足成功标准时，它依赖 Memory 中关于"过去什么算成功"的记录来进行参照。例如，在代码生成任务中，Memory 存储了"用户历史上接受过哪类改动"、"哪些边界条件曾导致测试失败"，Evaluator 在验证新生成结果时可以参考这些历史模式。

**Memory 作为 Harness 的基础设施**。在 Agent Harness 的七层架构中，Memory 系统（L4）与上下文工程（L3）、工具系统（L2）、执行引擎（L1）形成完整的依赖链。L4 Memory 提供知识准确性的保障（知识编译范式：写入时做对，读取时直接用），L3 上下文负责压缩和成本控制，L2 工具系统负责安全可控，L1 执行引擎负责双循环稳定性。没有 L4 Memory，L3 的压缩缺乏判断基准——无法区分"应该保留的核心知识"和"可以被丢弃的临时信息"。

## 遗忘的战略价值：为什么记忆系统必须会丢失信息

Agent Memory 系统设计中最反直觉的原则是：**系统必须会遗忘，而且必须有策略地遗忘**。这不仅是性能优化的需要（存储空间、检索效率），更是系统保持可塑性和正确性的必要机制。

**持续积累的幻觉陷阱**。一个只积累不遗忘的记忆系统，表面上看"信息最全"，实际上是在积累误解。 当一条旧信念从未被新信号挑战时，它会随着每次被调用而被强化——即使它是错的。这就是"Confirmation Bias"的系统化版本：历史记录越多，系统越倾向于认为"过去每次都这样判断，说明这个判断是对的"。但事实可能是：这条判断只在过去的特定前提下成立，前提早已改变。

**遗忘顺序与信息保真度损失**。研究表明，在反复的压缩-摘要-再压缩循环中，信息按照以下顺序丢失：Tone（语气）→ Context（上下文）→ Boundary Conditions（边界条件）→ Exceptions（例外情况）→ Temporal Constraints（时序约束）^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md]。对于需要精确边界判断的任务（代码生成的边界 case、合同审查的例外条款），这种损失是致命的。

**有策略遗忘的三种类型**。第一种是**降权遗忘**：长期无引用的条目降低权重（LLM Wiki V2 的自动遗忘机制），bug 记录快衰减（因为 bug 可能已修复），架构决策慢衰减（因为架构决策的影响更持久）。^[raw/articles/karpathy-llm-wiki-v2-2026.md] 第二种是**替换遗忘**：当新信号与旧信念冲突且新信号具有更高权威度或更新时效性时，旧信念被新信念替代，旧版本作为历史证据保留在 Timeline 层（GBrain 的 Compiled Truth + Timeline 双层结构）。 第三种是**主动遗忘**：被后续信号反复否定的旧 belief、高度情境依赖且低泛化的细节、已被更高层抽象吸收的底层 event——这些不是被动降权，而是主动标记为"已失效"。

**遗忘作为系统健康指标**。一个有策略遗忘机制正常工作的 Memory 系统，其遗忘行为本身携带着有价值的信息。一个条目被遗忘的频率和原因反映了知识的新鲜度和适用性；某个记忆在特定上下文被反复检索但从未被更新，说明该上下文可能是稳定的；两个冲突记忆在同一个上下文中被同时检索，说明该上下文可能存在未被发现的概念边界问题。

**Raw + Derived 双层约束防止过度遗忘**。过度遗忘（系统变"钝"）和过度积累（系统变"乱"）是记忆系统的两个极端。^[raw/articles/memory-vs-rag-agent-memory-systematic-framework.md] GBrain 的双层架构和上下文工程三种 Memory 方案对比实验中 RAG 与 MSA 的对比，都说明高质量的记忆系统需要 Raw 层和 Derived 层相互约束：Derived 层提供压缩后的语义视图，Raw 层提供可回溯的证据基础。遗忘决策需要同时满足两个条件：Derived 层的压缩版本仍然可用（不丢失关键语义），Raw 层的原始版本仍然可查（不丢失可验证性）。

## 子页面
- [[concepts/agent-memory-lifecycle-philosophies|生命周期与架构哲学]] — Raw vs Derived、遗忘机制、MemGPT、五架构哲学、治理层、评估
## Related Concepts
- [[concepts/hermes-agent]] — Hermes Agent closed-loop learning mechanism (Memory Nudge / Skill Nudge / Self-Improving flywheel)
- [[concepts/harness-engineering-framework]] — Harness Engineering six-layer structure (state memory layer)
- [[concepts/agent-memory-system-design]] — Wiki Query page, Agent Memory System design guide
- [[entities/agent-harness-architecture|Agent Harness 架构]]
- [[entities/claude-code-source-architecture|Claude Code 源码拆解：从启动到多 Agent 扩展层]]
- [[entities/agent-architecture-harness-new-backend|Agent架构关键变化：Harness正在成为新后端]]
- [[entities/agent-engineering-principles-architecture-practice|Agent 原理、架构与工程实践]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]
## 相关实体
- [[entities/three-rag-architectures-classic-graph-agentic|三种 RAG 架构对比：Classic、Graph、Agentic]]

- [[concepts/agent-backend-unification|Agent 与后端统一架构]]
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/architecture-data-foundations-for-ai-powered-search|Architecture & data foundations for AI-powered Search]]
- [[entities/agent-memory-architecture|Agent Memory 架构本质]]
- [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]]
- [[entities/gbrain|GBrain]]
- [[entities/context-engineering-three-memory-paradigms-comparison|上下文工程 - 三种Memory方案对比]]

## 新增关联实体
- [[entities/lesecretairedefernand-co-en-tech-explicit-vs-implicit-in-the-age-of-intelligence]]
- [[entities/self-evolving-agents-survey-papersagent]]

## 关联实体

**上游依赖**:
- [[entities/agent-harness-architecture]] — 提供基础理论/方法
-  — 提供基础理论/方法
- [[entities/claude-code-source-architecture]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-architecture-harness-new-backend]] — 具体应用场景
- [[entities/agent-engineering-principles-architecture-practice]] — 具体应用场景
- [[entities/ai-agent-engineer-capability-map]] — 具体应用场景

**平行协作**:
- [[entities/agent-memory-architecture]] — 替代/补充方案
- [[entities/thin-harness-fat-skills]] — 替代/补充方案
- [[entities/gbrain]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
