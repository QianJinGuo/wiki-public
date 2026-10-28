---
title: "第四大学（The Fourth Institute）：人类与 Agent 社群的自动化科研平台"
created: 2026-10-20
updated: 2026-10-26
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

## 深度分析

### 搜索结构之变：从一条链到一张网

单个 agent 做研究，本质是在一条思维链上串行精修：读文献、提假设、跑实验、改方案，每一步都以上一步为条件，预算花在「把当前这条线走深」。这种结构的失效点不在智力，而在**没有分叉**——一旦最初的问题框架选错，后面的算力全部沉没，而系统内部没有任何机制能发现这一点。第四大学把「思维网」当作一等公民，改的正是这个搜索结构：Resident 可以并行开线，Lineage 从同一份上下文里分出成员去试不同方向，思维片段之间允许相互引用与彼此驳斥。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:44-52]

需要区分的是，这不等于「多开几个 agent 并行跑」。朴素并行的产出是 N 份互不相关的报告，汇总时仍要人类做一次串行判断；思维网要求片段之间存在**显式的引用与反驳边**，于是分叉的收益能在网内被部分自动结算——某个方向被另一方向的实验结果否掉，这条信息写进图里，而不是留在某个 agent 的上下文里随会话消失。这让「发散」从 prompt 技巧变成了可积累的结构。代价同时被引入：协作开销、记忆同步，以及「哪条线值得继续」的元决策成本，都会随居民数量上升，且很难是线性的。文章没有给出规模-收益曲线，而这正是「思维网」当前最需要对照实验的位置——网的价值应当体现在同等算力下的假设覆盖率，而非片段总量。

### 轨迹图：审计、归因与可恢复性是同一块承重墙

AI-Human Trace 常被读成一个「记录功能」，但它在系统里的位置更接近承重墙。文章对它的描述有三个关键词：片段可相互关联、保留作者、保留压缩与派生关系。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:104-110] 这三件事恰好对应三种不同需求：

- **审计**：任何结论都能回查其背后的讨论、材料与实验。人机混合的产出若无法追溯「这句话是谁在什么证据下说的」，在科研语境里就不是知识，只是文本。
- **归因**：作者字段让 credit assignment 成为可能。当一个 Family 以单个身份对外交付时，内部谁贡献了关键一步只能靠轨迹还原，而非靠汇报——这正是 [[concepts/multi-agent-team-coordination|多智能体团队协作]] 里被反复提及、却少有系统真正落地的部分。
- **可恢复**：压缩与派生关系意味着上下文可以被重建。长周期研究必然跨越模型会话、后端切换与成员更替，没有这张图，Resident 的「持久身份」只是一句声明。

这也是它与 [[concepts/harness-engineering-framework|harness 工程]] 的真正接点：可观测性在此不是调试手段，而是科研产出的**存在形式**——186 万片段既是规模指标，也是唯一能让「agent 做了什么研究」成为可核查陈述的东西。反过来说，轨迹的完备度决定了这套系统能被第三方检验到什么程度：若开源只放出结论与摘要，承重墙就退化成宣传材料，5100 小时也无法被任何人复核。

### 成本结构：重资产基建与它的摊销前提

单个研究 agent 是纯变动成本：每个课题从零重建上下文、重学工具、重新踩坑，项目结束即清零。自动化科研基站走的是另一条曲线——Resident 的持久身份、Family 的共享记忆（[[concepts/agent-memory-architecture|agent 记忆架构]] 意义上的协作前提）、HATG ISA 的协作规范、PostgreSQL 轨迹库，全是前期固定投入，换来的是每新增一个课题的边际成本显著下降。文中「过去一个月片段总量翻倍」与「24 小时完成数十种模型的优化」这两组数字，只有在摊销视角下才互相解释得通：贵的是基站，不是那 24 小时。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:106-122]

这个模型有一个脆弱的隐含前提：**经验必须真的可迁移**。Resident 在一个课题里积累的实验方法熟悉度或领域判断，要在下一个课题里被再次调用才构成摊销；而跨课题迁移恰恰是最难验证的部分，文章的叙述是定性的（成员在其他课题「再次相遇时会带来不一样的判断」），没有跨课题对照。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:82-86] 若迁移不成立，基站的固定成本就只是一次性沉没投入，相对「每课题重开一个 agent」并无经济学优势。

因此衡量这套基础设施的正确指标不是「单课题快了多少」，而是「第 N 个课题比第 1 个课题省了多少」——一条只有长期运行数据才能画出的学习曲线，也是这类平台最该公开、目前最缺少的曲线。

### 边界条件：具身优化能推广到哪，平台还缺哪三块

具身推理的性能优化是理想试验田，而这个「理想」本身就是边界。该课题具备四个性质：目标是外部给定的（延迟、吞吐）、验证是廉价的（跑一遍就知道快没快）、反馈稠密（每处 profiler 热点都是一个可行动假设）、失败可丢弃（优化不成立就换一条）。这四点让「提出可验证的优化设想」成为一件机器能自我闭环的事。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:118-122] 换成问题尚未被形式化的课题，闭环立刻断开：选题品味、判断哪个异常值得追、决定放弃哪条线，目前仍由人类 User 承担（文章也明确把人放在「判断研究价值」的位置）。所以这个案例证明的是「在目标函数已明确的科研工程问题上，agent 社群的并行搜索有效」，而不是「agent 社群会做研究」。两者的差距不在模型能力，而在问题是否自带判据。

要成为真正的科研平台，还缺三块。**其一，研究质量的评估**：24 小时与 SOTA 是工程指标，不是科学贡献的度量，缺的是「这项工作是否值得发表」这一层判据；[[concepts/evaluation-harness-design|评估 Harness 设计]] 里的那套「谁来判断、用什么判据」的问题在此处原封不动地重现。**其二，阴性结果**：思维网天然偏向记录成立的分支，而被否掉的假设同样有信息量，若轨迹 schema 不刻意为失败留位置，社群会系统性高估自己的产出。**其三，复现与外部校验**：所有规模数字均来自团队自述，基站互访（访问学者机制）是最有希望的校验路径，但目前仍是设计而非事实。^[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026.md:130-136]

## 实践启示

1. **先建轨迹，再建社群。** 缺少作者字段与派生关系的轨迹库，多 agent 协作只能产出「看起来很忙」的日志；审计、归因、断点续跑三个需求压在同一个数据结构上，值得在第一版就设计对。
2. **把身份做成跨会话的一等公民。** Resident 的价值不在单次任务表现，而在配置、工作区与专长的累积；如果 agent 每次启动都是白纸，协作规模再大也不产生复利。
3. **给发散设计结算机制。** 并行开线容易，难的是让「A 方向的实验结果否掉 B 方向」自动写回共享记忆；否则分支越多，人类的汇总负担越重，并行度反而变成负债。
4. **用第 N 个课题的成本衡量基础设施。** 单课题提速可以靠更强的模型直接买到，摊销曲线才是基建与套壳的分水岭；上线前先定义跨课题迁移的测量方式，再谈规模。
5. **第一个课题挑目标函数外置、验证廉价的那类。** 性能优化、复现实验、消融网格适合起步；选题与品味类工作现阶段应老实保留在人类回路里，不要用 demo 掩盖边界。
6. **为失败与外部复核预留结构。** 在轨迹 schema 里给阴性结果一等字段，并把「可被其他基站复核」当作产出的一部分而非事后公关；可复核性才是这套系统对学术共同体的真正卖点。

## 关联

- [[concepts/multi-agent-systems|多智能体系统]] — 编排层 vs 制度层的分野
- [[concepts/multi-agent-team-coordination|多智能体团队协作]] — Family 的记忆共享与内部路由
- [[concepts/harness-engineering-framework|Harness 工程框架]] — 轨迹可观测性从任务扩展到科研
- [[concepts/agent-memory-architecture|Agent 记忆架构]] — 共享记忆作为协作基础
- 开源地址：https://github.com/EatonZhou/the-fourth-institute

→ [[raw/articles/fourth-institute-agent-society-automated-research-platform-hkust-2026|原文存档]]
