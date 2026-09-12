---
title: "State Lake：火山引擎面向 Agent 时代的存储基础设施重构"
created: 2026-07-16
updated: 2026-09-12
type: entity
tags: [storage, agent-infra, state-lake, sandbox, artifact, byte-dance, volcano-engine, agent-platform]
status: verified
confidence: 0.9
provenance_state: extracted
sources: [raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> 火山引擎存储团队提出"从 Data Lake 到 State Lake"的范式转变，围绕 Sandbox Store、Artifact Store、Agent 观测&评测三大方向重新组织存储能力，支撑 Agent 时代的真实业务。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

## 存储范式演进

AI Workload 三阶段驱动存储从 Content Storage → State Storage → State Lake：^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

| 阶段 | 数据形态 | 核心命题 |
|------|---------|---------|
| Training | 海量 Dataset, TB/s 带宽 | 持久化、冷归档、分发 |
| Inference | KV Cache, Embedding | 短生命周期张量高速流转、秒级恢复 |
| Agent | 沙箱/记忆/Trace/产物 | 环境状态、可观测、闭环进化 |

## Storage Agent Infra 三大方向

### Sandbox Store

面向 Agent 沙箱运行环境，三者覆盖沙箱的**状态、共享、通信**三个面：^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

- **EBS（云盘）**：沙箱本地状态，快照/延迟加载/批量创盘/pause-resume
- **EFS（共享文件）**：多沙箱并行实验、共享数据集、POSIX 语义、3000万 IOPS
- **MQ LiteTopic（消息队列）**：会话级隔离与强顺序，百万级动态创建与释放

### Artifact Store

Agent 执行过程中的输入输出与任务产物持久化：^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

- **EFS**：强 POSIX 语义（3000万 IOPS、亚毫秒时延、数据流动降本）
- **TOS 对象存储**：海量弹性、按需付费
- **Agent Bucket**：亿级 Agent 原生存储桶，引入 ObjectSet 层级
- **SenseFlow**：多模态理解引擎，产物从"存下来"走向"被理解"
- **ContextBucket**：Agent 记忆与工作区统一底座
- **ADrive**：Agentic 智能网盘（双空间隔离、持久化、自然语言检索）

### Agent 观测 & 评测

TLS AgentLoop：观测数据接入 → Trace 调用链/Session 分析 → 评测集管理 → 持续优化闭环。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

## Storage Agent Family

存储产品自身的 Agent 化：TOS/EBS/TLS/EFS/MQ 各自建设智能助手能力，支持记忆进化（用户确认后沉淀为知识）。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

## 深度分析

### 为什么 Agent 会击穿 Training / Inference 的存储假设

Training 与 Inference 的存储设计共享一个前提：数据短命、单向、可丢弃——训练集读完即归档，KV Cache 随会话失效。Agent 同时打破这三条：环境有状态（沙箱文件系统、已装依赖、进程内存）、过程有痕迹（成链 Trace 与 Session）、产物可被反复取用。存储因此要在同一底座上承载环境状态、过程痕迹、任务产物三类异质数据，并支持暂停、恢复、跨沙箱共享与再检索；被经典范式刻意剔除的持久性与语义，反成 Agent 时代的核心命题——这也是 [[concepts/long-running-agent-architecture|长时运行 Agent 架构]] 与 [[concepts/agent-sandbox|Agent 沙箱]] 生命周期管理强耦合的原因。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

### State Lake：Data Lake 之外多出来的那一层

Data Lake 解决「海量原始数据沉下来且可批处理」，State Lake 解决「Agent 运行状态沉下来且可查询、可闭环优化」，差别在「可查询的活状态」：前者是事后写入的静态事实，后者是可快照、可挂载、可回滚的运行现场。State Lake 还直接嵌进 observe → evaluate → optimize 循环——Trace 与评测集不再是存储之外的旁路，而是底座要托住的一等公民；衡量标准也从容量转向状态与闭环，与 [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|从指标到闭环的 Agent 评测]] 的思路一致。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

### Sandbox Store 三件套：状态、共享、通信

沙箱是三件事的叠加：保存现场的执行环境（状态）、交换数据的工作节点（共享）、与编排层对话的会话实体（通信）。EBS 承载状态——快照回滚、延迟加载按需拉取镜像、批量创盘应对高并发创建、pause-resume 让长任务挂起而不销毁；后两项直接命中成本与冷启动痛点：挂起不必空转计费，按需加载免去启动时搬运整镜像。EFS 承载共享——POSIX 语义让多沙箱并行读写同一份数据集，3000 万 IOPS 撑住实验带宽。MQ LiteTopic 承载通信——会话级隔离防消息串扰，强顺序保证指令与回执因果一致，百万级动态创建释放对齐沙箱「来去如潮」的生命周期。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

### Artifact Store：从「存下来」到「被理解」

语义弱、体积小的对象原本交给对象存储即可，Agent 产物却要求存储参与理解：Agent Bucket 用 ObjectSet 层级为「亿级 Agent 各持一堆产物」建立归属模型；SenseFlow 让产物从被写入走向被读懂；ContextBucket 把记忆与工作区收拢到同一底座；ADrive 以双空间隔离区隔 Agent 与人的操作域，用自然语言检索把「找文件」升级为「问文件」。存储由此不只回答「东西在哪」，还要回答「这是什么、属于谁、能拿来做什么」，这与 [[entities/ai-agent-storage-curvine-eks-2026|EKS 上的 AI Agent 存储]] 的产物治理同源，也把 [[concepts/context-engineering|上下文工程]] 的对象从 prompt 扩到持久化产物。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

### 闭环与反转：TLS AgentLoop 与 Storage Agent Family

TLS AgentLoop 把观测接入、Trace 调用链与 Session 分析、评测集与评估器管理串成持续优化闭环；其价值不止「看得见」，更在于反向定义存储层该留什么——哪些状态要留、留多久、以什么粒度回放，都由闭环下游需求倒推。这与 [[entities/agent-observability-5-layer-architecture|Agent 可观测五层架构]] 互为补充，也是 [[concepts/agent-self-improvement-loops|Agent 自改进循环]] 转起来的前提。同时「Storage Agent Family」完成一次反转：存储产品自身成为 Agent，TOS/EBS/TLS/EFS/MQ 各建智能助手，把用户确认过的结论沉淀为长期记忆。^[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16.md]

## 实践启示

1. **按「状态生命周期」而非「文件大小」选型。** 沙箱现场、Trace、产物保留期与访问模式不同，混在一套存储里只会两头受制；应先分清哪些需秒级恢复、哪些可冷归档。
2. **把 pause-resume 与快照延迟加载当作成本杠杆。** 挂起替代销毁、按需加载替代整镜像搬运，闲置成本与冷启动时延同时下降。
3. **多沙箱并行实验先定共享层与通信层。** 共享数据的 POSIX 语义、消息的会话级隔离与强顺序需提前约定，否则并行规模一上来就翻车。
4. **产物存储预留「被理解」的接口。** 只做对象落盘会锁死多模态产物的价值，索引、解析与检索应与写入路径一起规划（参见 [[entities/agent-memory-architecture|Agent 记忆架构]]）。
5. **记忆与工作区共用底座，并保留知识沉淀通道。** 可参考 [[concepts/agent-memory-substrate-three-layer|Agent 记忆底座三层]] 与 [[entities/agent-memory-storage-engineering-practical-guide|Agent 记忆存储工程实践]]。

## 关联条目

- [[entities/agent-harness-engineering-survey-2026|Agent Harness 工程全景]] — Harness 工程中存储层的角色定位
- [[entities/cloud-agent-infrastructure-creaoai-state-code-credential-isolation-20260606|Cloud Agent Infrastructure]] — Agent 基础设施的另一种视角（状态/代码/凭证隔离）

## 退出

→ [[raw/articles/byte-dance-volcano-state-lake-agent-storage-infra-2026-07-16|原文存档]]
