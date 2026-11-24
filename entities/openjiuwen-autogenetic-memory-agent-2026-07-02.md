---
title: "OpenJiuwen AutoGenetic Memory — 华为开源自主生长Agent记忆引擎"
created: 2026-07-02
updated: 2026-11-14
type: entity
tags: [agent, memory, openjiuwen, huawei, agent-memory, open-source, swarm-memory]
sources: [raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OpenJiuwen AutoGenetic Memory — 华为开源自主生长Agent记忆引擎

> AutoGenetic Memory 是华为 openJiuwen 社区开源的 Agent 自主生长记忆引擎，采用 L0-L3 分层记忆架构、AutoDreaming 后台异步加工、MemoryTurbo 涡轮加速、Graph Memory 关系图谱与 Swarm 群体记忆等核心机制，在 LoCoMo 基准上实现准确率提升 15%、Token 消耗降低超 60%。^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

## 核心架构

### L0-L3 分层记忆体系

JiuwenMemory 设计了四层记忆架构，让信息从原始对话逐级抽象为结构化知识：^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

- **L0 原始信息层** — 完整保留对话历史与时间戳，支持回溯与全量分析
- **L1 摘要记忆层** — LLM 压缩单次会话关键结论，降低 Token 消耗
- **L2 结构化记忆层** — 拆分为情景记忆（事件/决策时间轴）与语义记忆（背景知识/技术细节）
- **L3 用户画像层** — 核心特征刻画（偏好、习惯、角色、结构化变量）

分层价值：信息密度逐级放大、偏好与技术细节分离、全量历史免加载、Token 成本降低。

### AutoDreaming（"梦中"异步记忆加工）

借鉴认知神经科学的记忆固化理论，将高成本记忆提取操作从在线路径剥离，放到后台定时离线异步完成：^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

- **浅睡** — 增量筛选
- **REM 阶段** — LLM 单遍完成提取与归类
- **深睡** — 语义去重、冲突消解后写入长期记忆
- 多重控本：守护进程定时触发、忙碌退避、断点续扫、批次封顶

### MemoryTurbo（前台/后台动能解耦）

前台对话产生"排气"（新增信息），后台飞轮旋转转化为"增压"（结构化记忆）。原始对话瞬间写入缓存层向量库完成更新，记忆提取在后台按算力负载异步调度。^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

- 用户感知时延降低 80%
- Token 使用量进一步降低 50%+
- 提取未完成时缓存层仍可检索

### Graph Memory（关系图谱记忆）

将对话、文档和结构化内容转化为可持续演进的记忆知识图谱，Agent 沿着实体和关系理解长期记忆。^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

- 实体与关系链路召回，解决纯语义检索遗漏
- Episode 保留来源溯源
- 跨会话信息组织成关系网络

### Swarm 群体记忆

基于蜂群协作范式，每个 Agent 独立积累个体记忆，同时将共享经验沉淀至组织级记忆池，新 Agent 直接继承已有知识。^[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02.md]

### 动态 Adapter 层（双维度解耦）

- **Plugin 维度** — 面向 Agent 平台（已支持 OpenClaw）
- **Provider 维度** — 面向记忆引擎（已内置 JiuwenMemory 和 Mem0）
- 记忆跨平台继承，避免被单一框架锁定

## 基准性能

- **LoCoMo 测评集**：JiuwenMemory 接入 OpenClaw，准确率提升 15%，Token 消耗降低超 60%

## 深度分析

### "自主生长"的三段机制

基因隐喻对应三个工程环节：复制（原始对话无损写入 L0 与缓存层，任何提炼可回溯）、校对与自然选择（AutoDreaming 浅睡增量筛选、REM 单遍提取归类、深睡语义去重与冲突消解，矛盾过时的片段被淘汰）、遗传（Swarm 把个体经验沉淀进组织级记忆池供新 Agent 继承）。关键在全程离线、增量、可溯源：守护进程定时触发、忙碌退避、断点续扫、批次封顶让 Token 开销线性可控，每条 L2/L3 记忆都回指原始会话，是[[concepts/memory-consolidation-decay|记忆固化与衰减]]的工程化表达。

### 结构化与巩固的分工

L2 把会话拆为情景记忆（事件/决策时间轴）与语义记忆（背景知识/技术细节），对应[[concepts/episodic-vs-semantic-memory-agent|情景与语义记忆之分]]，避免偏好被流水账覆盖；Graph Memory 再把孤立事实连成实体-关系网络，链路召回弥补纯语义检索的遗漏。巩固由两条管线分工：Dreaming 按睡眠阶段离线提炼整段会话的隐式结论，MemoryTurbo 用离心式语义聚类让小模型先按话题归组、大模型一组一次调用。在线路径只剩"写缓存 + 缓存层兜底检索"，实质是[[concepts/working-set-vs-long-term-memory|工作集与长期记忆分离]]的模式。

### 基准数字说明什么、不说明什么

LoCoMo 上准确率 +15%、记忆问答与添加 Token -60%、感知时延 -80%。强处：三者对照的是 OpenClaw 原生的扁平时间轴记忆，证明"分层+离线加工"优于流水账基线。局限：LoCoMo 只是长对话 QA 类基准之一（见[[entities/agent-memory-evaluation-landscape-taobao-survey|淘宝 Agent 记忆评估全景]]）；对照物是特定 harness 的原生记忆，而非 Mem0 等同类引擎的同口径成绩；时延与 Token 降幅主要来自缓存前置与离线批处理的架构设计，即使准确率不变也大概率成立。选型时应把质量收益与成本收益分开归因，参考[[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]做多基准验证。

### 范式坐标系中的位置

AutoGenetic Memory 属于"加工/巩固派"：差异化不在存储选型（sqlite/postgres/redis/chromadb 皆可），而在离线提炼管线，与[[entities/context-engineering-three-memory-paradigms|三种记忆范式对比]]中把记忆当作持续治理对象的主张一致。对比 Provider 维度内置的 Mem0（以即时抽取-更新为主），它押注批量巩固与图谱化组织；对比[[entities/hermes-agent-memory-system|Hermes Agent 记忆系统]]、[[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]]等以文件/上下文为载体的方案，它更数据库化、面向多 Agent 共享。最有辨识度的是 Swarm 组织级记忆池（呼应[[entities/jiuwenswarm-coordination-engineering|OpenJiuwen Swarm 协作工程]]）与 Adapter 双维度解耦——把记忆当跨平台基础设施而非框架 feature，与[[concepts/agent-identity-portability|Agent 身份可迁移性]]关心的是同一个"不锁定"问题。

## 实践启示

1. **把高成本记忆加工移出在线路径**：提取、去重、冲突消解放后台定时任务，前台只写缓存并靠缓存层兜底检索；用忙碌退避、断点续扫、批次封顶控成本。
2. **分层阻止信息互相覆盖**：至少分原始/摘要/结构化三层，情景（做了什么）与语义（是什么样的人）分开存，画像常驻顶层。
3. **关系图谱补语义检索的洞**：多跳问题纯向量召回必漏，显式建实体-关系图并保留 Episode 溯源，检索时沿链路扩展。
4. **记忆按基础设施设计**：借鉴 Adapter 双维度解耦，定义面向 Agent 平台与面向记忆引擎两个抽象接口，换平台不丢记忆、换引擎不重写集成。
5. **质量与成本分开归因**：复现收益时分别测多基准准确率与 Token/时延，并消融缓存层与离线管线各自的贡献，勿被单一基准数字带偏。
6. **多 Agent 团队上组织级记忆池**：各 Agent 独立积累、定期沉淀共享经验，新成员直接继承画像与问题解决路径，免去冷启动。

## 相关资源

- 开源仓库：https://gitcode.com/openJiuwen/agent-memory/
- openJiuwen 官网：https://www.openjiuwen.com/

## 相关交叉链接

- [[entities/jiuwenswarm-coordination-engineering|OpenJiuwen Swarm 协作工程]] — 同一社区的 Agent 协作框架
- [[entities/agent-memory-architecture|Agent 记忆架构]] — Agent 记忆系统设计模式
- [[entities/agent-memory-architecture-ruofei|若飞 Agent 记忆架构]] — 分层记忆视角对比
- [[entities/hermes-agent-memory-system|Hermes Agent 记忆系统]] — 记忆引擎实现对比
- [[entities/state-of-memory-in-agent-harness-mem0-2026|Agent Harness 记忆现状]] — Mem0 等同类引擎
- [[entities/agent-memory-evaluation-landscape-taobao-survey|淘宝 Agent 记忆评估全景]] — 记忆评估体系
- [[entities/ai-agent-memory-systems|AI Agent 记忆系统综述]] — 记忆系统综合对比
- [[entities/context-engineering-three-memory-paradigms|三种记忆范式对比]] — 范式级对比
- [[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]] — 分层记忆设计对比

→ [[raw/articles/openjiuwen-autogenetic-memory-agent-2026-07-02|原文存档]]
