---
title: "Agent Teams 协作机制：从 ReAct 到七机制团队设计"
created: "2026-08-31"
updated: 2026-08-31
type: entity
tags: [agent, multi-agent, collaboration, teams, orchestration, harness-engineering, organization]
sources:
  - raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026
confidence: 0.75
provenance_state: inferred
---

# Agent Teams 协作机制：从 ReAct 到七机制团队设计

千问AI平台（蒋泽林/林曜）从两个月 Agent 开发实践出发，以第一性原理拆解 ReAct 循环，横评 7 篇多 Agent 协作论文，提出 7 机制 Agent Team 设计框架。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## 第一性原理：Agent = ReAct + 工具 + 观察

Agent 本质是 Thought → Action → Observation 循环的实现（pi-agent 核心约 600 行代码）。三环节对应：思考（模型基座决定智商）、行动（工具集决定手脚）、观察（环境反馈决定感知）。工程团队着力点在行动和观察——提供更好的工具、返回更结构化的执行结果。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

大模型无状态：每次调用独立前向计算，参数不因调用改变。人学会 Java 会物理性改写突触，大模型跑完一件事权重一字节不变。"上下文即记忆"是当前唯一可靠工程路径。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## 当前 Leader-Worker 架构不足

主流框架（CrewAI、AutoGen、MetaGPT）的 Leader 本质是调度器，缺少方案讨论、共识达成、动态重规划。对比现实团队五维度差距：Leader 角色（分发器 vs 资深专家）、任务下达（直接拆分 vs 讨论共识）、Worker 交互（隔离 vs 自由沟通）、进度管理（无 vs OKR）、方案变更（无审查 vs Leader 审查）。核心缺陷：Worker 之间完全隔离，缺少 P2P 横向信息流。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

AgentTeams（阿里 AgentScope 开源）是工业界工程完成度最高的 Manager-Workers 实现：Kubernetes 控制面 + Higress AI Gateway + Matrix 通信总线 + MinIO 共享文件。但"通信通道有了，协作语义没有"——没有方案讨论、OKR 协商、分歧仲裁、集体复盘、团队演化。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## 五阶段生命周期链（文献横评）

作者将多 Agent 协作研究串成五阶段链：分类与框架 → 角色与流程 → 目标管理 → 经验积累 → 团队演化。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

- **① 分类**（arXiv 2501.06322）：五种组织形态（Flat/Hierarchical/Team/Society/Hybrid），没有普适结构，灵活交互场景 Flat 更优
- **② 分工**：MetaGPT（ICLR 2024）SOP 编码 + Agent-Oriented Planning（ICLR 2025）动态拆解，互补但共同盲点：Worker 不参与目标制定
- **③ 目标**：OKR-Agent（arXiv 2311.16542）层级递归 OKR，但单 Agent 递归分解，Worker 无协商权
- **④ 经验**：Experiential Co-Learning（ACL 2024）个体经验库（0.43→0.73），但未上升为团队资产
- **⑤ 演化**：Meta-Team（arXiv 2605.29790）三层协作演化 +6.6%，但初始手工 MAS 9 测试 6 个不如单 Agent；EvoChamber（arXiv 2605.11136）CoDream 五阶段循环，消融实验：去掉协作进化后 20 Agent = 1 Agent

**关键结论：多 Agent 的价值 100% 来自协作机制本身，堆 Agent 数量本身不产生任何价值。**^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## 七机制 Agent Team 设计

### 7.1 Leader 重定义：资深专家兼管理者

四个特征：深度参与方案制定（有大致路径）、与 Worker 讨论后再执行（共识）、具备兜底能力（亲自接手）、负责方案变更审查（防局部最优）。Leader 与 Worker 应有能力差（更强模型、更长 context）。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.2 启发式管理：激发者非命令者

命令式描述→保守产出；鼓励式引导→深度创造性。机理：RLHF/DPO 对齐训练中"合作鼓励"语境对应高质量回复，"命令否定"语境对应保守回复。Agent 的沟通风格直接映射到 token 概率。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.3 讨论→共识→执行

三阶段协议：Leader Proposal → Worker Feedback → Consensus。执行层约束前置到规划阶段。Agent Team 中三 phase 可异步并行秒级完成。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.4 Worker 横向通信

三种模式：主动广播、被动查询、求助升级。需引入"熟人网络"、"技能索引"、"通信预算"约束防止全连接指数爆炸。A2A/ANP/Matrix 已提供通信底座，缺上层协作设计。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.5 OKR 目标与进度管理

Team OKR 由 Leader + 所有 Worker 共同讨论产出；Worker OKR 认领并拆解。里程碑触发、偏离预警、KR 达成即关闭。关键：Worker 必须参与 OKR 制定，因为他们才知道执行层真实约束。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.6 岗位要求与团队宗旨

显式 JD：能力要求、Skill 池、质量标准、汇报关系、SLA。岗位驱动能力而非能力决定岗位。Mission（为什么存在）→ 核心宗旨（做事原则）→ OKR（本季度做什么），三者是长期到短期的连续统。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

### 7.7 集体复盘与团队演化

Leader 主持、Worker 参与，输出三类资产：方法论、协作模式（team playbook）、反模式。借鉴 Meta-Team 三层设计：任务中微复盘 → 阶段交付中盘 → 任务结束总复盘。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## Agent Autonomy 分级

L1 Copilot → L2 Task Agent → L3 ReAct Agent → L4 Team Agent → L5 Autonomous Organization。当前 L3→L4，Meta-Team/EvoChamber 标志 L4→L5 过渡。^[raw/articles/agent-teams-collaboration-mechanism-jiangzelin-qianwen-2026.md]

## 相关实体

- [[entities/agentteams-和-claude-tag-都进入群聊模式是新范式还是新叙事|AgentTeams 与 Claude Tag 群聊模式]] — 工程实现维度（基础设施/Matrix/A2A），本文补充协作机制维度
- [[entities/agent-orchestration-multi-agent-systems|多 Agent 编排系统]] — 编排层控制面（Step Functions/审批门），本文聚焦组织设计
- [[entities/agent-productivity-paradox-collaboration-bottleneck|Agent 生产力悖论：协作瓶颈]] — 诊断问题，本文给出解决方案
