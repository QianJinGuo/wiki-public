---
title: "第四大学（The Fourth Institute）：人类与 Agent 社群的自动化科研平台"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [multi-agent, agent-society, automated-research, hkust, open-source, memory]
sources: [raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026]
confidence: 0.75
---

# 第四大学（The Fourth Institute）：人类与 Agent 社群的自动化科研平台

## 一句话定位

香港科技大学郑光廷、马松辰团队开源的「第四大学」不是「让 agent 模仿人类科研流程」的 agent 流水线，而是一座给 agent 建的**城市**：常驻身份（Resident）、可嵌套的协作群体（Lineage/Family）、统一协作指令集（HATG ISA）与持久轨迹库，目标是让人类与 agent 社群**共同**产出有判断价值的创新。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 问题设定：入场券只有 5000 张

文章引 Karpathy 的观察开场——全球大约只有 5000 人真正见过顶尖 AI 的能力，即让数千个 agent 长时间协作、昼夜不停做科研。于是问题变成：一个带几张 GPU 卡和几个模型账号的普通研究者，怎么参与进来？团队的答案借了人类制度史：先为探索「安一个家」，再出发——大学、学科、实验室都是这么来的，agent 也需要 Agent Society。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 核心设计：从思维链到思维网

第四大学的判断是「创新不住在一条思维链里，而住在许多思维链的分叉、交汇与重组中」。因此它把单个 agent 的线性思维扩展成一张可生长的**思维网**：思维片段可相互引用、彼此驳斥、跨研究路线重组。载体是三层组织原语：

- **Resident（居民）**：跨会话延续的常驻 agent 研究员，保留配置、工作区、研究轨迹，甚至拥有工作偏好与审美偏好（偏爱严谨复现还是大胆猜想），目的是摆脱「人类说一步、它走一步」的被动。
- **Lineage**：把多个 Resident 组织成围绕课题分工、交流发现的协作群体，也能从已有上下文分出新成员去探索不同方向。
- **Family**：更紧密的协作单元，成员**共享记忆**、对外以单个 Resident 身份接收请求再内部路由；可无限嵌套，构成复杂社会层级的基础。类比公司部门：用户只跟「经理」说话，团队组建与记忆同步由系统完成。

组织规范由 **HATG（Human-Agent Trajectory Graph）ISA** 承担：用固定指令集保证每项协作有章可循，但不规定研究内容。这套「身份 + 群体 + 指令集」的分层，是 [[concepts/multi-agent-systems|多智能体系统]] 里少见的「制度层」实现——多数框架只做到编排层。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 实例化：自动化研究基站的三个部件

一个具体实例对应人类实验室的三件套：

| 实验室要素 | 第四大学对应物 |
|---|---|
| 研究员 | Agent Society（人类 User + AI Resident；Codex、Claude Code 等后端通过适配接入为统一身份的 Resident） |
| 知识与仪器 | 异构资源（知识/记忆文件、代码、实验材料、算力，乃至 Resident 本身都可被发现与引用） |
| 规范与保障 | 协作规范与执行设施 = Multi-Agent 操作系统（插件与后端适配持续扩展） |

开放性体现在「可以把跑在自己设备、用自己框架的 agent 接进基站协作并同步轨迹」，像派一位研究员去外单位访学，不被平台边界锁死。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## AI-Human Trace：把科研变成可回溯的轨迹图

系统以 PostgreSQL 保存人机交互与 agent 执行轨迹：输入、模型输出、工具调用被切成可相互关联的片段，片段的作者、执行记录、压缩与派生关系一并保留，任何结论都能回查其背后的讨论、材料与实验。规模上，截至发稿已积累 **186 万余条思维与执行片段、93 万余条交互事件**，平台上 agent 累计开展 **5100+ 小时**研究与开发；过去一个月片段总量翻倍。团队计划陆续开源这些自动化科研轨迹，供社区研究人机协作方式。这条设计把 [[concepts/harness-engineering-framework|harness 工程]] 的「可观测、可回放」从单任务扩展到长期科研过程。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 第一个真实课题：具身推理的自动化性能优化

团队把自己的研究方向（软硬协同设计）作为首个课题：具身基础模型（VLA / WAM 及其结合）与端侧、边缘算力设备都在碎片化发展，至今没有统一推理框架承接。做法是让 agent 团队面对一个 PyTorch 实现先完整运行、观察时间花在哪，再按模型与设备组织成员，提出一个个可验证的优化设想。结果是 **24 小时**内完成数十种模型到多种 GPU 平台的推理性能优化，部分结果达到 SOTA；后续还将开源面向物理智能的 Agentic OS。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 判断：这是「科研基础设施」而非「科研 agent」

值得记住的三点区分：

1. **制度优先于编排**：Resident 的持久身份、Family 的记忆共享与嵌套、ISA 的协作规范，解决的是「agent 之间如何长期共处」，而不是单次任务如何拆解。
2. **人类的位置被显式设计**：人类以 User 身份带着自己的知识库加入，个人知识既启发 Agent Society，也被其成果反哺——这与 [[concepts/agent-memory-architecture|agent 记忆架构]] 中「共享记忆是协作前提」的判断互为印证。
3. **可验证性靠轨迹而非声明**：186 万片段 / 5100 小时的量化，使「agent 做了什么研究」成为可审计数据。

风险也清楚：文中所有规模数字与 SOTA 声明均来自团队自述，尚无第三方复现；「思维网」的增益缺少对照实验；Family 的共享记忆在长期运行下的污染与漂移未被讨论。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md]

## 关联

- [[concepts/multi-agent-systems|多智能体系统]] — 编排层 vs 制度层的分野
- [[concepts/multi-agent-team-coordination|多智能体团队协作]] — Family 的记忆共享与内部路由
- [[concepts/harness-engineering-framework|Harness 工程框架]] — 轨迹可观测性从任务扩展到科研
- [[concepts/agent-memory-architecture|Agent 记忆架构]] — 共享记忆作为协作基础
- 开源地址：https://github.com/EatonZhou/the-fourth-institute

→ [[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026|原文存档]]
