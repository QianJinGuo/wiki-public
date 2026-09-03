---
title: "上下文工程"
created: 2026-06-11
updated: 2026-08-13
type: concept
tags: [agent, harness, prompt-engineering, context-engineering, memory, context-window, context-compaction, prompt-caching, retrieval-augmented]
description: "上下文工程：Prompt 设计 → 上下文窗口管理 → 信息密度优化的范式演进"
sources:
  - raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026
---

# 上下文工程

**上下文工程（Context Engineering）** 是 2025 年兴起的一场系统性工程学科，比 [[concepts/prompt-engineering-fundamentals|Prompt Engineering]] 范围更大、层次更高。它要回答的不再是"怎么写好一条指令"，而是"怎么让 Agent 在正确的信息边界内持续做对"——一个需要编排工具、记忆、压缩策略、执行控制协同发力的系统级挑战。

本概念页汇聚 wiki 中 **[[entities/claude-code-context-engineering-anthropic-thariq|Anthropic Claude Code 团队]]**、**[[entities/codex-context-engineering-lastwhisper-thinking-in-context|LastWhisper Codex 解读]]**、**[[entities/anthropic-prompt-caching-claude-code|Anthropic Prompt Caching 九原则]]**、**[[entities/agent-harness-architecture-design-production-guide|Agent Harness 七层架构]]**、**[[entities/claude-code-prompt-source-analysis|Claude Code Prompt 源码解析]]**、**[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 工程实践]]**、**[[entities/agentcore-harness|AgentCore Managed Harness]]** 等 8 个核心实体的核心洞察，形成对上下文工程的系统性认知框架。

## 核心定义

### 从 Prompt Engineering 到 Context Engineering

**Prompt Engineering** 解决的核心问题是：模型是否听得懂你在说什么？优化重心是单次交互中的指令表达——如何写好一条 system prompt、如何few-shot 示例排列组合、如何在一条消息里把话说清楚。

**Context Engineering** 要解决的问题则是：模型是否拿到了足够且正确的信息？优化重心从"怎么说"转移到"给什么看"。这要求开发者不仅设计 Prompt 本身，还要管理**上下文窗口（Context Window）**的内容结构、加载顺序、生命周期和更新策略。

Anthropic Claude Code 团队的 Thariq Shihipar 首次系统化表述了这一转变：上下文管理的本质不是塞更多信息进窗口，而是**在约束条件下做最优的信息供给设计**——包括哪些信息该进窗口、哪些该压缩、哪些该存到外部、压缩后的信息如何保证有效检索。

> **定义**：上下文工程是在 [[concepts/harness-engineering-framework|Harness Engineering]] 框架下，围绕 LLM 上下文窗口构建的**信息架构体系**，涵盖上下文窗口管理（Window Management）、信息密度优化（Information Density）、上下文压缩（Context Compaction）和检索增强上下文（Retrieval-Augmented Context）四个核心维度。其目标是保证模型在任意执行时刻，都能在正确的信息边界内做决策。

### 与 Prompt Engineering 的明确分工

[[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程]] 指出：PE 负责"说什么"（指令的语义表达），CE 负责"看什么"（信息的选择与组织）。两者的边界清晰：

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|---------------------|
| **核心问题** | 怎么说才听得懂 | 给什么看才做得对 |
| **优化对象** | 指令文本质量 | 信息结构与供给策略 |
| **时间尺度** | 单次交互 | 跨会话、多轮、带状态 |
| **工程重点** | Few-shot 排列、角色设定、格式约束 | 窗口分层、压缩策略、记忆管理 |

## 关键维度

### 维度一：上下文窗口管理（Window Management）

#### 分层缓存架构

[[entities/anthropic-prompt-caching-claude-code|Anthropic Prompt Caching 工程实践]] 揭示了 Claude Code 的核心工程约束：**Prompt Caching = 前缀匹配**。只要下次请求的前缀跟上次一样，就能复用计算结果。这意味着上下文窗口管理的第一原则是**越不容易变的东西越往前放**。

Claude Code 的分层结构：

| 层级 | 内容 | 缓存范围 |
|------|------|----------|
| **第1层** | 静态系统 prompt + 工具定义 | 全局（所有 session 共享）|
| **第2层** | CLAUDE.md 文档 | 项目级（同一项目内共享）|
| **第3层** | Session 上下文 | 会话级（单次会话内共享）|
| **第4层** | 对话消息 | 逐轮增长（每轮只新增最后一条）|

这个分层体系的核心洞察是：**不可变性是缓存友好的前提**。Prompt 不可变 → 静态前缀稳定，跨 session 复用；工具集不可变 → 主对话缓存链不中断；模型不可变 → 缓存资产不因模型切换而蒸发；消息可变 → 动态信息通过消息流转，不污染缓存前缀。

#### 缓存失效的灾难链效应

分层缓存架构中，任意一层出问题都会引发连锁反应：

- **第1层**（系统 prompt + 工具定义）出问题 → 所有 session 全局失效
- **第2层**（CLAUDE.md）出问题 → 同一项目所有会话失效
- **第3层**（Session 上下文）出问题 → 单会话内所有轮次失效
- **第4层**（对话消息）出问题 → 当前轮次后所有轮次失效

[[entities/anthropic-prompt-caching-claude-code|Anthropic 经验]] 指出了一个极难排查的典型 bug：Python 3.7+ 的 dict 保持插入顺序，但 `json.dumps()` 默认不保证排序。如果工具定义在序列化时顺序打乱，每次请求前缀实际都在变，但肉眼完全不可见。这种 bug 需要在序列化层强制 `sort_keys=true` 或使用有序数据结构。

#### Prompt 排列优先级策略

[[entities/claude-code-prompt-source-analysis|Claude Code Prompt 源码解析]] 揭示了优先级策略树（P0-P4）的覆盖语义：

| 优先级 | 类型 | 规则 |
|--------|------|------|
| **P0** | Override | 硬覆盖，其他所有 prompt 全部失效 |
| **P1** | Coordinator | coordinator mode 开启时，主线程变为调度者 |
| **P2** | Agent | mainThreadAgentDefinition 设置时；proactive mode 下追加而非替换 |
| **P3** | Custom | 用户 `--system-prompt` 传入 |
| **P4** | Default | 最终兜底 |

**静态/动态分离**是关键设计：静态规则缓存（不频繁变化的部分），动态规则每次会话更新。两者有明确 boundary 划分，使得缓存失效范围清晰可控。

#### Cache-Safe Forking（压缩的学问）

当 Context 填满后需要压缩另起调用时，[[entities/anthropic-prompt-caching-claude-code|Anthropic]] 观察到：如果另起调用用了不同的 system prompt → 从第一个 token 就跟主对话缓存对不上，整个缓存积累的资产就全部浪费了。

**解决方案**：压缩请求必须用跟主对话完全一样的 system prompt、user context、工具定义，把主对话消息作为历史带上，在末尾追加压缩指令作为新 user message。这样压缩请求复用主对话缓存链，新增成本只有最后那条压缩指令本身。

### 维度二：信息密度优化（Information Density）

#### 语义 vs. 潜在压缩的不对称

[[entities/codex-context-engineering-lastwhisper-thinking-in-context|LastWhisper Codex 解读]] 提出了一个深刻的能力不对称问题：

**应用层 — Semantic Compression**：
- 应用开发者能实现的压缩 = **语义重构**（LLM 摘要）
- 始终是原始信息的有损投影
- 推理微观逻辑、注意力分布模式在文本摘要中不可避免丢弃
- 通用模型预训练目标并非上下文压缩

**基础设施层 — Latent Compression（推测）**：
- 厂商在模型**隐空间**对高维向量做压缩
- 文本是模型内部状态的**低维投影**
- 隐空间含更丰富信息结构，高维空间有损压缩上限更高

但有重要工程问题：向量表征与模型架构深度耦合，模型升级 → 向量表征失效 → 需重新适配。更可能的实现是厂商有信息优势的 Semantic Compression——模型内部状态访问、海量真实用户数据、专用 Fine-tuned 压缩模型。

#### Append-only vs. Context Editing 两种流派

[[entities/codex-context-engineering-lastwhisper-thinking-in-context|LastWhisper]] 对比了 Codex 和 Claude Code 的不同流派：

| 流派 | 代表 | 核心动作 | 收益 | 代价 |
|------|------|----------|------|------|
| **Append-only** | OpenAI Codex | 状态变化时追加 | 前缀稳定，缓存命中 | 上下文膨胀 |
| **Context Editing** | Anthropic Claude Code | 原地修改/剪枝 | Token 精简 | 前缀破坏，工程复杂 |

**Codex 的 Append-only 原则**：当 Agent 状态变化（切换审批模式、切换工作目录），不修改已发送消息，而是追加新消息。这类似 Event Sourcing 模式——"State is a projection of events"。收益是前缀始终不变 → 最大化 Prompt Cache 命中，因果链保留。但代价是上下文膨胀和噪声影响性能。

#### 作用域上下文组装

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 提出了作用域上下文组装模式：把指令拆到不同作用域里（组织级、用户级、项目根目录、父目录、子目录），Agent 会根据当前所在的位置动态加载对应的规则。这既能保持全局一致，又能允许局部有差异，适用于 Monorepo、多语言项目。

#### 渐进式披露（Progressive Disclosure）

OpenAI 团队的经验（[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 引用）：AGENTS.md 只有 ~100 行充当"目录页"，指向仓库里的详细文档。**核心原则：不是把所有信息塞进上下文，而是让模型在需要时能找到信息**。

### 维度三：上下文压缩（Context Compaction）

#### 阶梯式上下文压缩

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 的 L3 层定义了阶梯式上下文压缩策略：

- **L0**：保留关键信息
- **L3 以下**：逐步删除低分内容

这种分层策略避免一刀切截断导致的语义断裂，同时会话隔离（独立生命周期 + LRU 淘汰）确保多租户场景下上下文不会相互污染。**Token 消耗降低 52%**，阶梯压缩提供了可预测的 Token 成本上限。

#### 渐进式上下文压缩模式

[[entities/agent-harness-architecture-design-production-guide|同一实体]] 描述了渐进式压缩的具体做法：

- 新的对话尽量保留细节
- 稍旧的内容做轻量总结
- 再往前的就逐步压缩，甚至折叠成很短的摘要

可以理解为**越久远的信息，保留得越粗**。适用场景是对话轮次比较多（比如 20～30 轮以上）的任务。权衡点是压缩一定是有损的——信息在一轮轮总结中会丢失，如果后面又需要这些细节，Agent 可能会"编"而不是承认不知道。

#### 知识编译（Knowledge Compilation）

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] L4 层提出了知识编译模式：将知识单元转化为 QA 对，是降低幻觉率的关键——结构化的 QA 格式比自由文本更不容易被 LLM 误用。**幻觉率从 30% 压至 5% 以下，准确率 95%+**。

这与直接 RAG（直接 embedding 原文）形成鲜明对比——记忆系统的核心挑战不是检索精度，而是如何将知识转化为模型可靠使用的形式。

#### Claude Code 的 5 层渐进式压缩管线

VILA-Lab 对 Claude Code 的逆向工程（[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 引用）揭示：Claude Code 记忆**完全基于文件系统**（CLAUDE.md + JSONL 日志），没有向量数据库、没有 Embedding，采用 **5 层渐进式压缩管线**（裁剪 → 截断 → Auto-Compact 全量摘要）。**流程状态细节在 Auto-Compact 阶段丢失**是当前架构的一个已知取舍。

#### VikingMem 的 Token 效率发现

VLDB 2026 的 VikingMem 研究（[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 引用）发现：**16.82% Token 留存得分 75.80 vs 朴素 RAG 100% 留存仅 63.81**。结论是"**更少 Token + 更智能组织 > 全量保留**"。这验证了信息密度优化的核心假设——并非越多上下文越好。

### 维度四：检索增强上下文（Retrieval-Augmented Context）

#### 三层记忆架构

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] L4 层定义了完整的三层记忆架构：

| 层级 | 存储 | 容量 | 延迟 | 用途 |
|------|------|------|------|------|
| **短期记忆** | 内存 | ~128K tokens | <1ms | 即时上下文 |
| **中期记忆** | 本地 DB (SQLite/PostgreSQL) | ~10MB | <10ms | 跨会话持久化 |
| **长期记忆** | 向量库 | 无限制 | ~100ms | 语义检索 |

#### 四类 Memory 分层

[[entities/claude-code-prompt-source-analysis|Claude Code Prompt 源码解析]] 定义了 Memory 的四类分层：

| 类型 | 内容 | 写法 |
|------|------|------|
| **user** | 角色/目标/知识水平/偏好 | 直接描述 |
| **feedback** | 对 Agent 工作方式的反馈 | 规则 → Why → How to apply |
| **project** | 事实/决策/动机/截止时间/事故背景 | 事实 → Why → How to apply |
| **reference** | 看板/dashboard/Slack/Linear 入口 | 直接描述 |

**不进 Memory**：代码结构、git 历史、修 bug recipe、CLAUDE.md 内容、临时任务状态。这种边界划分防止记忆系统过载。

#### 持久化指令文件模式

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 提出的持久化指令文件：放一个项目级的配置文件（`CLAUDE.md`），每次会话自动加载。里面写清楚构建命令、测试方式、架构规则、命名约定这些内容。文件跟着代码仓库走，而不是靠人每次复制粘贴。适用场景是需要在多个会话里反复处理同一个代码库。权衡点是有维护成本——这个文件需要跟着项目一起更新，一旦过时，反而会误导 Agent。

#### 记忆整合模式（Dream Consolidation）

[[entities/agent-harness-architecture-design-production-guide|同一实体]] 描述了 Claude Code 源码中提到的 `autoDream` 机制：加一个后台整理机制，在空闲时定期做清理——去重、删旧、重组结构，让记忆保持干净、可用。相当于给 Agent 做一次"垃圾回收"。

### Anthropic 四类失败模式

[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 引用了 Anthropic 的四类失败模式研究：

| 失败模式 | 描述 | 解决方案 |
|---------|------|----------|
| **One-shot Syndrome** | 复杂需求在单个上下文窗口内完成，窗口超过 40% 填充率后质量快速衰退 | 上下文窗口 Sweet Spot < 40% |
| **Premature Victory Declaration** | Agent 完成部分工作就宣布结束，核心功能未实现或验证 | 引入端到端验证（Puppeteer MCP 截图） |
| **Premature Feature Completion** | Agent 认为功能已实现但未做端到端测试，部署后关键路径不通 | Browser Automation 自动化验证 |
| **Cold Start Problem** | 多次会话间缺乏持久化记忆，新会话需大量 Token 重新理解项目 | progress.md + 持久化记忆体系 |

**共同根源**：Agent 缺乏外部的结构化约束（Structured Constraints）和反馈机制（Feedback Mechanisms）。**根本能力缺陷**：*"Agents are incapable of accurately evaluating their own work"*。

### 维度五：插件抽象的可扩展性光谱（Daisy Hollman）

Anthropic Claude Code 插件与 Agent 团队负责人 Daisy Hollman（NDC Copenhagen 2026 演讲，InfoQ 整理）给出了判断上下文供给原语的核心标准：**是否只在相关时才往上下文窗口注入内容**——"不为你不用到的东西付费"的零开销原则。所有定制化都在与工作空间竞争，整个代码库根本放不进去，必须智能挑选相关上下文。^[raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026.md]

| 抽象 | 注入方式 | 扩展性瓶颈 |
|------|---------|-----------|
| MCP 服务器 | 工具名+描述+schema 常驻系统 Prompt | 20 个服务器×15 工具即占满窗口 |
| 工具搜索 | 先加载工具名+搜索工具（套娃） | 百万工具仍用光上下文；工具名描述性决定触发 |
| Skill | 惰性系统 Prompt：正文按使用付费 | description 始终加载，10 万 Skill 照样塞满 |
| 子 Agent | 描述展开到独立上下文窗口 | Skill 是上下文内的，子 Agent 是上下文外的 |
| Hooks | 事件触发脚本，上下文外运行 | 唯一真正可扩展：不匹配不注入 |

**MCP 的扩展性代价**：MCP 是给系统 Prompt 增加工具调用 schema 的 JSON 协议，服务端拥有认证权、传输无关，适合需要跨客户端可移植的集成；但如果已有 CLI，创建一个解释如何使用该 CLI 的 Skill 可能比 MCP 服务器好得多。Monorepo 里一千个 MCP 服务器全部加载进上下文时，每个工具的名称/描述/schema 都要放系统 Prompt，20 个服务器×15 个工具就占满大半窗口——MCP 没法在没有帮助的情况下扩展。工具搜索（先加载名称+给 Claude 一个搜索工具的工具）只是缓解，且名字越有描述性越可能被搜到。^[raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026.md]

**KV 缓存硬约束**：next-token 预测要求前缀 token 完全一致，否则预测成本贵 10 倍；每次工具调用都重排上下文会带来极其庞大的 token 消耗。Cursor 早期在 Cursor Rules 上按 LRU 缓存思路（根据当前任务换入相关规则、驱逐不相关规则）很快发现成本高得离谱——这个问题比单纯 LRU 缓存微妙得多。^[raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026.md]

**Hooks 是唯一真正可扩展的抽象**：在上下文窗口之外运行脚本，根据输出协议决定是否注入。post tool use hook 相当于 Agent 的"红色波浪线"——在错误发生的时刻给轻推（类型检查/linting/CLAUDE.md 合规标记），而不是等模型回头编译时才意识到，反馈循环越紧密 Agent 表现越好。**CLAUDE.md 反模式**：无条件在上下文开头注入大量文本的抽象，使用端成本太高，限制同时只能激活 5-10 个插件。**记忆（Memory）不是上下文工程基元**：记忆是模型策展的文本文件，超大规模软件工程的未来必须在"上下文工程"与"记忆"之间做出明确区分，两者需独立运作。^[raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026.md]

**多 Agent 工作流的规模化实践**：Git worktrees 起步最简单（每个会话独立 worktree，互不踩脚，演讲者常驻 26 个 A-Z 各配一个长期维护的命名 Agent）；Agent 团队用 Send message 工具让会话间对话（团队领导委派/平级共享）；/loop 是给模型用的 cron 定时工具（每 10 分钟自动唤醒，防止模型在任务完成前"睡着"）；自动模式（多轮模型+分类器筛查危险操作）让 20-30 个 Agent 同时跑成为可能，成本 +10-40%；Claude Agents（Fleet View）用轻量分类器把 10 个会话摘要显示在一屏（一周合并 ~1000 PR）。2026 年的核心转变：从"把信息输入模型"转向"把信息从模型输出给用户"——人的注意力才是系统中最小的盒子。^[raw/articles/claude-code-no-magic-context-engineering-primitives-daisy-hollman-2026.md]

## 演进路线

### 三次工程重心迁移

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 和 [[entities/agentcore-harness|AgentCore]] 都记录了同样的三次范式跃迁：

| 阶段 | 时间 | 核心问题 | 工程重点 | 隐喻 |
|------|------|----------|----------|------|
| **Prompt Engineering** | 2022-2024 | 模型是否听得懂你在说什么？ | 优化指令表达 | 写好一封邮件 |
| **Context Engineering** | 2025 | 模型是否拿到了足够且正确的信息？ | 优化信息供给 | 给邮件附上正确附件（Tobi Lutke） |
| **Harness Engineering** | 2026 | 模型是否能在真实执行中持续做对？ | 优化运行控制系统 | 造一辆好车 |

**核心引用**（[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]]）：

- Ryan Lopopolo（OpenAI）：*"Agents aren't hard; the Harness is hard."*
- Mitchell Hashimoto（HashiCorp）：*"Every time you discover an agent has made a mistake, you take the time to engineer a solution so that it can never make that mistake again."*

### 从 Context Engineering 到 Harness Engineering

[[entities/agentcore-harness|AgentCore]] 给出了更精准的阶段定义：**Harness = 模型之外的一切**——编排逻辑、执行环境、工具连接、状态管理、身份认证、可观测性。模型负责思考，Harness 负责让思考落地。

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 的七层金字塔中，Context Engineering 位于 **L3 上下文工程** 和 **L4 记忆系统**：

| 层级 | 名称 | 核心问题 |
|------|------|---------|
| L1 | 核心执行引擎 | 双循环、多模型、稳定性 |
| L2 | 工具系统 | 标准定义、权限、沙箱、MCP 生态 |
| **L3** | **上下文工程** | **隔离、压缩、成本优化** |
| **L4** | **记忆系统** | **短期/中期/长期记忆、低幻觉 RAG** |
| L5 | 自主决策引擎 | 目标管理、自主规划、自学习 |
| L6 | 多 Agent 协作 | 任务分配、共识、冲突解决 |
| L7 | 垂直行业应用 | 医疗、法律、金融、研发 |

### ETCLOVG 七层架构（CMU-Amazon 2026 Survey）

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 引用了 CMU-Amazon 的 ETCLOVG Taxonomy，其中 **C - Context & Memory** 正是上下文工程的主战场：

| 层级 | 名称 | 核心问题 |
|------|------|---------|
| E | Execution Environment | Agent 代码在哪跑、什么沙箱约束 |
| T | Tool Interface & Protocol | 外部能力如何描述、发现、调用 |
| **C** | **Context & Memory** | **模型每步能看到什么** |
| L | Lifecycle & Orchestration | 控制流如何读写状态 |
| O | Observability & Operations | traces、成本、失败、可靠性信号 |
| V | Verification & Evaluation | 任务 traces 如何变成评估和回归反馈 |
| G | Governance & Security | 跨模型、系统、组织层的行为约束 |

**关键发现**：Context & Memory 和 Governance & Security 在开源生态中最为薄弱，是未来工程投入的重点方向。

## 实践启示

### 1. 从缓存友好的角度设计 Prompt 布局

[[entities/anthropic-prompt-caching-claude-code|Anthropic]] 的第一条工程经验是：**Prompt Caching = 前缀匹配**。所有设计的核心约束都围绕这一事实。

具体做法：
- **越不容易变的东西越往前放**：系统 prompt + 工具定义置顶，对话历史置底
- **不要在 Prompt 里嵌入时间戳**：每秒都在变，缓存废掉
- **工具定义顺序必须固定**：任何位置变动（哪怕调整两个 Tool Definition 顺序）都使后续所有 Token 缓存失效
- **强制排序所有结构化数据的序列化**：JSON 序列化统一使用 `sort_keys=True`

### 2. 区分"宪法"和"日常行政"

[[entities/anthropic-prompt-caching-claude-code|Anthropic]] 的心智模型：**prompt 是"宪法"，消息是"日常行政"**。宪法不天天改，日常行政事务走消息通道。

过时信息（时间戳、文件状态）不要去改 prompt，而是用 `<system-reminder>` 标签塞进 user message 或 tool result 里。架构原则：**prompt 是「不可变的基础设施」，消息才是「流动的信息层」**。

### 3. 用子 Agent 做缓存边界隔离

[[entities/anthropic-prompt-caching-claude-code|Anthropic]] 策略：主对话自始至终用同一模型。需要小模型时用子 Agent，它有独立上下文和缓存，不污染主对话缓存链。这与微服务架构中的"通过 API 通信而非共享数据库"原则完全对应——共享缓存就像共享数据库，是强耦合的根源。

### 4. Context Reset vs. Context Compaction

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 区分了两种策略：

- **Context Compaction（压缩历史继续跑）**：保留历史摘要，填满后压缩继续
- **Context Reset（直接换一个干净上下文）**：通过工作交接实现"清空包袱、重新出发"的效果

### 5. 监控 Prompt Cache 命中率作为 uptime 级别指标

[[entities/anthropic-prompt-caching-claude-code|Anthropic]] 内部把 Prompt Cache 命中率当作 `uptime` 级别的指标监控，一旦下降就触发 oncall 告警。告警响应 checklist：

- 最近一次部署是否改了 prompt？
- 工具定义顺序最近是否有变更？
- 是否有账号池混用导致缓存隔离失效？
- session 平均长度是否突然下降（说明缓存链根本没建起来）？

### 6. 用确定性生命周期钩子执行必须做的事

[[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构]] 的黄金法则：**把确定性的事情交给系统，把判断的事情交给模型**。格式化代码、校验输入、重新加载配置——这些事情必须每次都做，而且不能出错，所以不应该依赖模型的记忆，而应该由系统强制执行。

### 7. 知识编译比直接 RAG 更能降低幻觉

[[entities/agent-harness-architecture-design-production-guide|同一实体]] 的 L4 层经验：不要直接将原始文档注入长期记忆。先通过知识编译（实体抽取 + QA 对生成）将非结构化知识转化为结构化单元，再存入向量库。结构化 QA 格式比自由文本更不容易被 LLM 误用，幻觉率可从 30% 压至 5% 以下。

### 8. 企业级项目的隐性知识问题

[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 揭示：典型企业级 Java 项目中存在大量**未编码的架构约定**——价格精度处理、链路高频变更区、全局配置类的近百处引用——这些信息存在于资深工程师的直觉中，却从未以机器可读的方式写入代码库。

解决方案：**建立机器可读的架构知识库**，将原来存在于老工程师脑子里的隐性知识转化为代码库可读取的规范文档（`.md` / JSON / 代码注释），让 Agent 能在运行时访问。

### 9. 评测平台的核心价值：用实验回答架构之争

[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 的 7 维评测平台发现：原始 token 消耗 + 工具调用解释 agent 成功率方差 **R²=0.33~0.42**，而**验证反馈质量（Effective Feedback Compute）达到 R²=0.94~0.99**。结论：**"决定 AI 干活靠不靠谱的并非给它多少预算，而是检查做得多好"**。

### 10. 薄主会话原则

[[entities/harness-engineering-alibaba-java-case-study|阿里 Harness 实践]] 的杜学友提出了三条铁律：

1. **主会话只听 dispatcher**：禁止自己 Read `phases/*.md` / `evidence.json`
2. **职责隔离**：每个 agent 的可用工具严格受限
3. **上下文 ≤8K**：只加载 CLAUDE.md + 触发规则 + 最近一条 dispatcher 指令

## 相关实体

- [[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程 —— Anthropic 团队的工程实践]]
- [[entities/codex-context-engineering-lastwhisper-thinking-in-context|Codex 上下文工程 — Prompt Layout + Append-only + Latent Space Moat（LastWhisper 解读）]]
- [[entities/anthropic-prompt-caching-claude-code|Prompt Caching 工程实践 — Anthropic Claude Code 经验总结]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]]
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]]
- [[entities/harness-engineering-alibaba-java-case-study|阿里工程师 Harness 工程化实践 (双案例合并)]]
- [[entities/agentcore-harness|AgentCore Managed Harness]]
- [[entities/agent-tools-research|微信文章：深度解析 Hermes Agent 如何实现'自进化'及其 Prompt / Context / Harness 的设计实践]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]
- [[entities/chromium-ai-coding-development-system|Chromium AI Coding 开发体系]]

## 相关概念

- [[concepts/harness-engineering-framework|Harness Engineering Framework]] — 上下文工程的上位框架
- [[concepts/prompt-engineering-fundamentals|Prompt Engineering Fundamentals]] — 上下文工程的相邻基础
- [[concepts/agent-memory-system-design|Agent Memory System Design]] — 三层记忆架构的具体设计
- [[concepts/claude-code-tool-design-evolution|Claude Code 工具设计演化]] — 工具定义与上下文管理的关系
- [[concepts/kairos-claude-code-paradigm|KAIROS — Claude Code 常驻协作范式]] — 上下文工程在实际协作中的应用

## 所属 MOC

- [[moc/layer-2-interaction|Layer 2 Interaction]]
