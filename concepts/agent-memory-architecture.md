---
title: "Agent 记忆架构"
created: 2026-06-11
updated: 2026-08-29
type: concept
tags: [agent, architecture, memory, llm, rag]
description: "Agent 记忆架构：工作记忆、长期记忆、情景记忆的系统设计与选型"
---

# Agent 记忆架构

Agent 记忆架构：工作记忆、长期记忆、情景记忆的系统设计与选型。本概念页汇聚 wiki 中多个相关实体的核心洞察，形成系统化的知识框架。

## 核心定义

**Agent Memory（Agent 记忆）** 不是简单的对话历史存储，而是解决"哪些过去可以继续影响未来"这一核心工程问题的**跨会话治理子系统**。它与以下概念有明确边界：^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

| 概念 | 定位 | 关键特征 |
|------|------|---------|
| **上下文窗口（Context Window）** | 当前工作集 | 临时、可丢弃；目标让本轮推理可解 |
| **Session** | 当前会话连续性 | 对话历史+工具调用+阶段性计划；短期状态 |
| **Profile** | 消费视图 | 低维快照（名字/角色/语言偏好）；不足以建模完整记忆 |
| **Policy** | 外部规则 | 权限/安全/合规/预算；memory 只能引用不能改写 |
| **Memory** | 跨会话可更新历史 | 持续存在、可更新、可审计、影响未来决策 |

> **一句话定义：** Memory 保存的是可跨时延续并影响未来决策的结构化历史——带来源、作用域、时间权重和可修正性的历史对象，而不是"把聊天记录再存一份"。 ^[raw/articles/agent-memory-architecture-essence.md]

**Memory 的本质不是存储，而是治理。** 存储只回答"东西放哪儿"，Memory 要回答的是：什么值得写入、以什么身份写入、在什么范围内有效、什么时候降低权重、和旧记忆冲突时听谁的、被污染后怎么回滚、用户能不能查看修改删除。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

---

## 记忆类型体系

### 三类工程记忆

Agent 记忆不止用户偏好，至少需要覆盖三类差异化记忆，它们的更新机制完全不同： ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

| 类型 | 内容 | 更新机制 | 示例 |
|------|------|---------|------|
| **任务记忆** | 需求进度、被否方案、当前版本、未完成承诺 | 随任务进展实时更新 | "方案 A 被否决，方案 B 进行中" |
| **环境记忆** | 仓库结构、团队规则、API 约束、部署方式、CI 特点 | 相对稳定，但边界模糊需定期同步 | "数据库只读" |
| **自我记忆** | 工具稳定性、失败推断、子代理启用判断 | 最不可靠，需要置信度标注和定期复核 | "工具 X 在长文本场景容易超时" |

把这三类混在同一个 memory 字段里，后续的冲突检测、衰减处理、权限管理都会变得不可维护。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

### 认知科学启发的记忆分层

所有成熟的 Agent 记忆系统都不约而同地采用了类似人类认知的分层架构： ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

- **持久记忆层（Persistent Memory）**：
  - **语义记忆（Semantic Memory）**：事实、决策、约定 → 无衰减，永久保留
  - **程序记忆（Procedural Memory）**：技能、习惯 → 频率衰减，不常用则淡化
- **工作记忆层（Working Memory）**：当前 session 的对话缓冲，session 结束后归档或丢弃

### 基本记忆单元：六维度

每条记忆不应只是文本片段，而应包含六个计算维度： ^[raw/articles/agent-memory-architecture-essence.md]

| 维度 | 含义 | 适用类型 |
|------|------|---------|
| **内容（Content）** | 这条记忆说了什么 | 全部 |
| **类型（Type）** | event / assertion / belief / constraint / commitment | 全部 |
| **置信度（Confidence）** | Agent 对这条记忆有多确信 | belief, commitment |
| **来源（Source）** | 用户表达/行为推断/环境观察/Agent生成 | 全部 |
| **作用域（Scope）** | 它在什么上下文下成立 | 全部 |
| **时间与衰减（Time & Decay）** | 产生时间、上次确认时间、衰减权重 | 全部 |

**类型详解**：
- **event**：发生了什么（高确定性事实）
- **assertion**：用户明确声明了什么（可直接推翻）
- **belief**：Agent 推断出来的（需通过新证据逐步修正）
- **constraint**：不可违反的边界（权威来源通常不在 memory 子系统内部）
- **commitment**：Agent 做出但尚未完成的承诺

---

## 四大建模对象

面向完整工程实现，记忆的建模对象分成四类： ^[raw/articles/agent-memory-architecture-essence.md]

| 模型 | 覆盖内容 | 示例 |
|------|---------|------|
| **用户模型** | 偏好、风险偏好、沟通习惯、决策模式 | 用户从"抵触TypeScript"到"主动要求重写"的转变轨迹 |
| **任务模型** | 被否决的方案、已确认的结论、当前真版本、未完成的承诺 | 事情推进到了哪一步 |
| **世界模型** | 操作环境：仓库结构、API约束、系统边界、组织规则、数据新鲜度 | 大量"个性化错误"本质是没注意到环境已变化 |
| **自我模型** | 试过什么、哪条路径失败、哪个工具在什么场景下不稳定、暂定假设 | Agent 不是在学习，只是在重复犯错 |

**意图不是被单独存在某个字段里的东西**。它是这四层模型长期耦合后浮现出来的上层能力——就像跟了三年的助理"懂你"，不是因为背了一本偏好手册，而是同时理解你的脾气、项目进度、组织环境和他自己的能力边界。 ^[raw/articles/agent-memory-architecture-essence.md]

---

## 三主链路：写入-管理-读取

记忆系统不是容器，而是 **write–manage–read** 三条链路的闭环。 ^[raw/articles/agent-memory-architecture-essence.md]

### 写入（Write）：预算分配

记忆写入本质是 **decision under budget**——存储预算有限，检索预算有限，未来注意力更有限。写入要决定的是：哪些信息值得获得对未来决策的影响力。 ^[raw/articles/agent-memory-architecture-essence.md]

关键洞察：
- 不能只看"这条信息有没有价值"，要看它相对于已有记忆的**边际价值**
- 与已有信念冲突的新信号（如一直保守的用户突然要求尝试 alpha）= 高价值信号，值得优先写入
- **行为证据通常比口头表态更值得写入预算**——"用户说过不喜欢ORM"是 assertion；连续三次提供ORM方案后又手写SQL是可以提炼为belief的行为模式

**写入链路最容易犯的错是把假设写成事实**。在长周期 Agent 里尤其危险：一个 Agent 误判写入，下一个 Agent 读到可能把它当成已验证的事实。再跑几轮，错误假设就长成了团队共识。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

### 管理（Manage）：最被低估也最决定长期质量

管理链路至少处理五件事： ^[raw/articles/agent-memory-architecture-essence.md]

1. **整合（Integration）**：把碎片信号聚合成结构化信念（蒸馏在这里发挥价值，但只是手段之一）
2. **冲突处理（Conflict Resolution）**："以最新为准"是偷懒的蒸馏思维。更合理的做法是保留矛盾，建模为"此维度上的偏好是情境依赖的"，在读取时根据当前情境选择
3. **衰减与遗忘（Decay & Forgetting）**：不能忘的系统会被旧判断拖死。遗忘不是 bug，是防止过拟合现实的必要机制
4. **来源追踪（Provenance Tracking）**：没有 provenance，Agent 无法判断自己的信念有多可信，也无法在出错时回溯责任链
5. **权限治理（Access Control）**：用户必须能查看、编辑、删除 Agent 的记忆

#### 遗忘机制的科学基础

遗忘是 Agent Memory 中最容易被忽视的能力。MemoryAgentBench 已经把 **selective forgetting** 当成一项能力来考核，LongMemEval 也把 knowledge updates 和 abstention 放进评估维度。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

成熟遗忘的特征：^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]
- 不是暴力删除，而是**版本感知、时间感知、依赖传播感知的退出机制**
- 删除的不是一段文本，而是一条**影响力链条**
- 失效后：仍可追溯，但不再主导当前行为

ai-memory 的 M8 衰减策略提供了一个精确的量化模型：
```python
score = salience · exp(-λΔt) + σ · log(1+access_count) · exp(-μ · days_since_access)
```
其中 `salience` 是初始重要度评分，`access_count` 是访问次数，`Δt` 是时间差，λ 和 μ 是衰减率。这个公式本质上是时间衰减和频率巩固之间的博弈——单纯的指数衰减会低估高频访问项的重要性，单纯的访问计数会忽略时间维度。 ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

### 读取（Read）：任务约束驱动

**传统做法的局限**：RAG 式的语义相似度召回（假设相关性由表面语义决定）。真正有价值的记忆调用往往是反直觉的——用户问"帮我写缓存方案"，最相关的记忆可能不是上次讨论缓存的对话，而是三个月前提到的黑五流量问题。 ^[raw/articles/agent-memory-architecture-essence.md]

**升级方向**：从语义相似召回，升级为**任务约束驱动的检索-推断耦合**：
1. 先由任务理解层判断"当前决策真正受什么约束"
2. 再由检索层去找对应记忆
3. 最后评估这些记忆在当前情境下的适用性

接口上：从 `retrieve(query)` 到 `read(task_context, belief_graph)` 的转变。 ^[raw/articles/agent-memory-architecture-essence.md]

---

## 存储方案：六大流派

当前 GitHub 上数十个 Agent 记忆项目可归为六大流派，每个流派代表一种设计哲学： ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

| 流派 | 代表项目 | 核心思路 | 典型规模 |
|------|---------|---------|---------|
| **向量记忆层** | mem0ai | 通用记忆层，LLM 提取 + 存储 + 检索事实 | **57K⭐**（最大社区） |
| **Wiki 编译派** | ai-memory | Session → LLM 总结 → Markdown wiki，Git 版本控制 | 467⭐ |
| **知识图谱派** | mnemon | 从对话中提取实体关系构建知识图谱 | 322⭐ |
| **会话历史派** | Letta / MemGPT | 完整 session 存储，支持 archival recall | 学术界主流 |
| **原始数据派** | obelisk, Hermes | 原始结构化数据直存 SQLite | 工程师倾向 |
| **仿生记忆派** | Anamnesis | 情景/语义/程序记忆 + 遗忘曲线 | 神经科学启发 |

### 核心争论：Wiki 编译 vs 原始数据

这是 Agent 记忆领域最核心的设计分歧——**信息压缩 vs 信息保真**。 ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

@QuantumTransf（Twitter）对 ai-memory 的质疑：
> 我没明白为什么要把 agent session 编译成 wiki。原始 session 本来就是结构化数据——messages、tool calls、tool results、files、subagents。直接放进 SQLite，就已经是一个很强的结构。而把它先总结成 markdown page，反而引入了一个不必要的中间实体：信息被压扁，因果链和引用关系要靠后续重建。

| 对比维度 | Wiki 编译模式 | 原始数据直存 |
|---------|--------------|------------|
| **人类可读性** | 极佳，Markdown 可浏览 | 差，需查询工具 |
| **信息保真度** | 有损，LLM 总结会丢失细节 | 无损，保留完整因果链 |
| **跨 Agent 互操作** | 任何能读 Markdown 的 Agent 都能理解 | 需标准化查询协议 |
| **存储成本** | 总结后体积小 | 原始数据量大 |
| **因果链追踪** | 需事后重建 | 天然保留完整时间线 |

**行业正在从"二选一"走向分层压缩**——保留原始数据的同时，按需生成多个压缩层级。原始数据提供信息保真度和因果链追踪能力，Wiki 摘要提供人类可读性和跨 Agent 互操作性。最佳实践是保留原始数据作为唯一真相源，其他表达层（wiki、图谱、向量）都是可选的派生视图。 ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

---

## 检索策略演进

Agent 记忆的检索能力经历了五代演进： ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

1. **关键词搜索**（FTS5 / BM25）—— 基础
2. **向量相似度**（embedding + cosine）—— 语义召回
3. **混合检索**（FTS5 + 向量并行）
4. **知识图谱邻居**（图遍历 + 关系推理）
5. **RRF 融合**（Reciprocal Rank Fusion）—— 当前最佳

**RRF 融合公式**：
```python
score = Σ(1 / (k + rank_i))  # k 通常取 60
```
将 FTS5 关键词结果、向量相似度结果、知识图谱邻居结果通过倒数排名融合。这比单一检索方式效果好得多，因为不同检索策略捕捉的是不同的相关性信号。 ^[raw/articles/agent-memory-storage-six-schools-quantumtransf-debate-frank.md]

---

## 架构模式与代表系统

### MemGPT：操作系统隐喻

MemGPT 的核心直觉是：把 LLM 的上下文窗口当作 RAM，外部存储当作磁盘。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

| OS 概念 | Memory 等价 |
|---------|-----------|
| RAM（工作区） | Context window |
| Disk（长期存储） | External Memory storage |
| Scheduling | Memory management: swap-in, swap-out, retain, compress, traceback, reorganize |

这个隐喻解决了混乱问题：
- **"为什么更大的上下文窗口不等于长期记忆？"** → RAM 再大也不是磁盘，增加上下文窗口只是增加了工作内存，存储管理机制不因 RAM 变大而自动建立
- **"为什么记忆必须分层？"** → OS 的内存管理本身就是分层的（L1/L2/L3 Cache → RAM → 虚拟内存 → 磁盘）
- **"为什么系统必须主动决定什么进入主上下文？"** → 上下文窗口是 RAM，RAM 的调度是由操作系统决定的

### Claude Code 七层记忆架构

Claude Code 的 7 层渐进式记忆管理系统借鉴人脑记忆分层原理，从毫秒级轻量清理到"做梦机制"巩固长期记忆，层层递进，成本递增，能力递增。 ^[raw/articles/claude-code-7-layer-memory-architecture.md]

| 层级 | 名称 | 核心机制 | 触发成本 |
|------|------|----------|---------|
| 第1层 | 工具结果存储 | 超过阈值时完整结果写到磁盘，上下文只放预览 | ~0 |
| 第2层 | 微压缩 | 时间基准清理、缓存微型压缩、API级上下文管理 | ~0 |
| 第3层 | 会话记忆 | 实时维护结构化笔记 | ~0 |
| 第4层 | 全压缩 | 9部分摘要生成 | 高（API调用） |
| 第5层 | 自动记忆提取 | 跨会话持久知识提取到 memory/ | 中 |
| 第6层 | 做梦机制 | 四阶段：标定→收集→合并→整理 | 高 |
| 第7层 | 跨代理沟通 | 分支代理模式 + SendMessage Tool | 中 |

**防御金字塔原则**：每层成本递增、能力递增，层层防护。尽可能防止下一层（更昂贵的）触发。 ^[raw/articles/claude-code-7-layer-memory-architecture.md]

### LangChain Memory

LangChain 提供了多种 Memory 组件实现：
- **ConversationBufferMemory**：简单缓冲当前会话
- **ConversationSummaryMemory**：摘要式压缩
- **VectorStoreRetrievedMemory**：向量检索召回
- **CombinedMemory**：多类型组合

LangChain 的 Memory 设计体现了模块化思路，但在 Management 层的冲突处理和遗忘机制上较为薄弱。

### 其他记忆架构模式

| 架构 | 核心 | 优势 | 成本/问题 |
|------|------|------|---------|
| **文件驱动** | Memory 作为可见、可编辑的外部文本 | 透明、可干预、可审计 | 自动演化能力弱 |
| **图驱动** | 关系 + 时间有效性 | 处理"同一对象不同时间状态" | 实现复杂度高 |
| **混合存储驱动** | 向量 + 图 + KV 组合 | 平衡召回/关系推理/时间变化 | 协调复杂度高 |
| **策略学习驱动** | 学习记忆管理策略 | 取代手工启发式规则 | 策略可解释性挑战 |
| **技能蒸馏驱动** | Memory 终点 = 可复用能力 | 最激进，上限最高 | 最危险：错误被高效固化 |

---

## 记忆生命周期

### 进化 = 修正 + 遗忘

**自我修正**：当 Agent 基于记忆做出了用户不满意的响应，这个负反馈应该**回溯到记忆层**判断问题在哪一层： ^[raw/articles/agent-memory-architecture-essence.md]

- 是检索召回错了？
- 是某条 belief 过期了？
- 是 belief 没错但被错误应用到了当前 scope？

只在回答层修补，却不修正上游假设 = 没有在学习，只是在打补丁。

**有策略的遗忘**什么该忘： ^[raw/articles/agent-memory-architecture-essence.md]
- 被后续信号反复否定的旧 belief
- 高度情境依赖且低泛化的细节
- 已被更高层抽象吸收的底层 event

> **核心命题**：死的不是经验本身，而是那些失去了更新机制的经验。Few-shot 示例、摘要、fine-tuned preference profile——它们并不天然低级。真正的问题是，一旦脱离了持续校正闭环，就从资产变成了惯性。 ^[raw/articles/agent-memory-architecture-essence.md]

### 记忆系统的评测维度

Memory System 的质量不能靠单一指标衡量，必须建立多维度评测框架： ^[raw/articles/agent-memory-architecture-essence.md]

| 维度 | 问题 | 测试方法 |
|------|------|---------|
| **检索召回率** | 能召回多少相关记忆？ | 构造已知记忆的查询任务，检查召回率 |
| **信念一致性** | 信念在相关上下文中是否保持一致？ | 相同上下文多次查询，检查置信度和表述是否漂移 |
| **压缩保真度** | 上下文压缩后，Agent 还能正确执行任务吗？ | 长任务中途触发压缩，对比压缩前后完成质量 |
| **遗忘效率** | 被遗忘的记忆确实不再影响 Agent 行为吗？ | 标记某条记忆为"待遗忘"，随后检查是否被应用 |
| **写入延迟** | 记忆写入的额外延迟是否影响交互体验？ | 测量有无记忆写入的响应时间差异 |
| **用户可审计性** | 用户能清晰理解记忆系统的状态吗？ | 用户调研——能否找到特定记忆并理解其来源 |

---

## 六大学派与设计哲学

Agent Memory 领域存在六个主要流派，代表不同的设计哲学和技术路线：

### 1. 向量记忆层（mem0ai 路线）

**代表**：mem0ai（57K⭐）
**核心思路**：通用记忆层，LLM 提取 + 存储 + 检索事实
**优势**：社区最大，接入最简单，语义召回能力强
**问题**：旧事实混淆、近义误召回、来源不清

### 2. Wiki 编译派（ai-memory 路线）

**代表**：ai-memory
**核心思路**：Session → LLM 总结 → Markdown wiki，Git 版本控制
**优势**：人类可读、可审计、可版本化，跨 Agent 互操作性好
**问题**：信息有损压缩，因果链需事后重建

### 3. 知识图谱派（mnemon 路线）

**代表**：mnemon、Graphiti、Zep
**核心思路**：从对话中提取实体关系构建知识图谱
**优势**：擅长关系推理、时间演化、多跳查询
**问题**：提取质量不稳定、图谱维护成本高

### 4. 会话历史派（Letta / MemGPT 路线）

**代表**：Letta、MemGPT
**核心思路**：完整 session 存储，支持 archival recall
**优势**：学术主流，理论框架完整
**问题**：太大容易污染上下文

### 5. 原始数据派（SQLite 直存）

**代表**：obelisk、Hermes Agent
**核心思路**：原始结构化数据直存 SQLite
**优势**：信息保真度最高，因果链天然保留，工程倾向
**问题**：人类可读性差，需查询工具

### 6. 仿生记忆派（Anamnesis 路线）

**代表**：Anamnesis
**核心思路**：情景/语义/程序记忆 + 遗忘曲线
**优势**：认知科学基础扎实
**问题**：实现复杂度高

---

## 治理层设计

Memory 的治理问题比存储问题更根本。治理层要回答的不是"如何获取信息"，而是： ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

- 这条记忆从哪来的？原始语句、摘要还是推断？
- 新旧信息冲突时，谁覆盖谁？
- 用户要求删除时，哪些衍生信息被连带失效？
- 哪些敏感信息不能跨场景穿越？
- 哪些内容仅是系统推测，不能伪装成事实？

**六个治理维度**：来源（Source）、权限（Permissions）、生命周期（Lifecycle）、置信度（Confidence）、影响范围（Impact Scope）、可撤销性（Revocability）

> **Memory 更像是操作系统，而不是数据库。** 数据库关心数据是否存在。操作系统关心资源如何被调度、隔离、继承、回收、审计和控制。 ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

---

## 多 Agent 记忆共享

当多个 Agent 协同处理同一个任务时，记忆共享问题变得尤为尖锐。核心问题不是"如何共享"，而是**如何在共享的同时保持边界清晰、问责可追溯、系统一致**： ^[raw/articles/agent-memory-architecture-past-influence-future-ruofei.md]

- **谁能看见什么？**
- **谁能修改什么？**
- **谁知道谁知道什么？**
- **一个 Agent 的错误记忆能否毒化整个协作网络？**

**多 Agent 记忆的核心难度**：不是把东西放在同一个地方，而是让不同的 Agent 对同一个过去形成**可协商的、隔离的、有问责的**访问结构。

**工程实践建议**：
1. **结构化交接文档代替共享内存**：通过显式的交接文档传递必要上下文，每个 Agent 的记忆边界通过交接点清晰划分
2. **记忆来源必须携带信任等级**：当 Verifier Agent 质疑 Worker Agent 的产出时，需要判断这个错误是来自 Worker 的记忆污染还是 Verifier 自己的记忆偏差
3. **定期跨 Agent 记忆一致性校验**：类似数据库的定期一致性检查

---

## 相关概念

- [[concepts/agent-memory-system-design|Agent Memory System Design]] — 记忆系统的详细设计指南
- [[concepts/agent-memory-lifecycle-philosophies|Agent Memory 生命周期与架构哲学]] — 记忆的治理、遗忘机制与多 Agent 共享
- AI Agent 记忆类型 — 工作记忆 vs 情景记忆 vs 语义记忆

## 相关实体

### 架构本质与理论
- [[entities/agent-memory-architecture-essence|Agent Memory 架构本质]] — 四建模对象、六维度记忆单元、write–manage–read 闭环
- [[entities/agent-memory-architecture-past-influence-future-ruofei|Agent Memory 架构解析]] — 若飞：Memory 作为控制面的治理视角
- [[entities/agent-memory-modular-framework|Agent Memory 模块化框架与评测]] — ICLR 2026 四组件框架与实验验证

### 代表系统深度解析
- [[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]] — 防御金字塔设计、做梦机制、分支代理模式
- [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|Agent 记忆存储方案深度洞察]] — 六大流派、RRF 检索融合、M8 衰减策略
- [[entities/hermes-agent-memory-system|Hermes Agent 记忆系统]] — 原始数据派的核心实践

### 评测与基准
- [[entities/memory-in-the-llm-era-iclr2026|Memory in the LLM Era]] — 论文：MemTree/MemOS 树状层次方法新 SOTA
- [[entities/memory-agent-systems-cobanov|memory agent systems cobanov]] — 9 个框架对比分析

### 工程实践
- [[entities/how-ai-agent-memory-works|AI Agent 记忆系统架构]] — 记忆系统工程化落地指南
- [[entities/context-engineering-three-memory-paradigms|Context Engineering 三记忆范式]] — 三种记忆范式的对比与应用

## 实践启示

### 设计原则
1. **先跑通 write–manage–read 全链路，而非追求存储容量** — 没有管理链路的存储是在积累垃圾，一个只有写入没有管理的记忆系统比没有记忆系统更危险
2. **从语义检索升级到任务约束检索** — 在检索层做最小改动，不要只调用 embedding similarity，在检索结果排序时加入上下文相关性评分
3. **建立信念置信度衰减机制** — 要在写入时就建立置信度和衰减权重，让系统主动淘汰低置信度记忆
4. **架构设计别从工具清单开始，而是从信息的生命周期开始** — 先定义信息类型，再决定每类信息的生命周期，最后选择匹配这个生命周期的存储方案

### 存储选型决策表
| 信息类型 | 更适合放哪里 |
|---------|------------|
| 项目长期规则 | AGENTS.md / CLAUDE.md |
| 已确认架构决策 | DECISIONS.md / ADR |
| 用户长期偏好 | memory store（带类型和版本）|
| 当前任务进度 | PROGRESS.md / 上下文窗口 |
| 完整事件日志 | event log / artifact |
| 未验证猜测 | progress observation，标记未确认 |
| 安全和权限规则 | policy 系统，memory 只允许引用 |

### 安全与治理
- **Memory 一旦可写，就要按持久化数据和执行上下文来对待**
- read-only 和 read-write store 要分开
- 共享 memory 默认只读
- 处理网页、邮件、第三方文档等不可信输入时，默认不要让它直接写长期记忆
- 每次写入要有版本，关键 memory 要能人工 review
- 用户能查看、修改、删除，被撤销的 memory 不能继续进入默认读取链路

### 评测转向
从"能不能 recall"到"能不能 update、能不能 abstain、能不能 handle drift、能不能 selective forget"。Memory 的难点从来不在容量，在治理。评测方向也应该从单一召回率转向多维度治理能力评估。

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
