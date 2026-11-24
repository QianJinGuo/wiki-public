---
title: "Anthropic Agent Platform 进化：三位高管深度对谈"
created: 2026-07-13
updated: 2026-11-13
type: entity
tags: [anthropic, agent, platform, claude-platform, agent-identity, agent-collaboration, scaffolding, managed-agent]
sources: [raw/articles/anthropic-agent-platform-evolution-three-executives]
confidence: 0.6
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Anthropic Agent Platform 进化：三位高管深度对谈

> **Background**：本文基于 Anthropic 三位高管（Jess Yann、Katelyn Lesse、Angela Jiang）的内部对谈整理，讨论 Claude Platform 过去半年在智能体基础设施、身份权限、智能体间通信、ROI 衡量和工程团队结构等方面的演化。

Anthropic 三位负责 Claude Platform 的高管——管理式智能体产品经理 Jess Yann、Platform 工程负责人 Katelyn Lesse、Platform 产品负责人 Angela Jiang——在半年的时间点上总结了智能体基础设施的变化方向。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 身份与权限层的演化

智能体的身份未来可能会和具体工作流本身分开，智能体自己拥有一份独立身份。趋势是智能体先听清楚用户想要的结果，然后说明自己需要访问哪些权限，用户在确认范围内授权，智能体给自己开一个服务账号去执行，用户可以随时审计。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 智能体间通信

团队在 Claude 管理式智能体的基础上搭了一层很薄的 MCP 服务器对外暴露，另一个智能体只需要知道怎么调用就能和这个智能体对话。多智能体协作不是单一策略，而是组合使用：互相竞争、互唱反调、顾问策略（模型想不明白就主动求助更聪明的伙伴）。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 脚手架变薄

模型变强后，团队开始把限制性强的框架部件逐步拿掉，脚手架正在变得越来越薄。脚手架变薄后反而出现一种「元脚手架」（未来可能被称作类似 Harness 的框架），负责在不同智能体和流程之间做协调编排。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## ROI 衡量

不要一上来就想全局智能化。合理的做法：个人 → 团队 → 跨团队流程三步走。先看一个人的工作速度能提升多少，再从个人推广到团队，再从团队推广到整体流程。衡量的重点应首先放在速度和生产力上。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 工程团队变化

人员构成没有太大变化——还是需要理解系统、知道运维、知道值班的那批人。区别在于每个人现在都被智能体大大增强了。以前技术负责人做系统设计，工程师领任务执行；现在几乎整个团队的每个人都能端到端搭建系统，再指挥自己的 Claude 完成具体工作。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 失败模式

依赖智能体带来的虚假超级独立感——一个人可以同时启动十个原型方案，让它们各自跑起来，最后挑最好的那个。但从系统整体看，把各自为战的产出重新协调到一起反而更难，容易出现蔓延式的失控。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

## 未来方向

未来智能体会更深入地嵌入组织内部，深到感觉不到自己在用某个工具。会出现一种共同的基底，用户只需要标记想要哪个智能体，随时启动、随时关闭，大量工作在看不见的地方自己完成——提前发现服务异常、自动排查、修复、准备 PR。更像一层看不见的操作系统，而不是一个个需要主动找的工具。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

Claude Platform 正在推的重点概念是「结果」：用户告诉智能体什么样的结果算好结果，给出一份评分标准，设定尝试次数。未来会接近用户告诉 Claude 想要什么结果、给出一个预算，然后放手不管的状态。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md]

→ [[raw/articles/anthropic-agent-platform-evolution-three-executives|原文存档]]

## 深度分析

### 平台战略：从卖 token 到收编抽象层

半年前 Claude Platform 还只是「调用推理、拿 token」的 API 加一些周边小工具，如今已把基础设施与脚手架工程的负担整体接走——这是一条清晰的「向上收编」路线。它的战略含义是：当平台把「结果」做成产品接口，用户只需给出评分标准与尝试预算、平台负责交付可验收的结果，竞争焦点就从模型质量转移到抽象层位置——谁替用户收编的工程负担越多，谁越难被替换。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:43] 在管理式智能体之上再暴露一层很薄的 MCP 服务器，则把每个 [[concepts/managed-agents-architecture|管理式智能体]] 都变成可被其他智能体调用的平台节点，平台边界由此从「模型 API」扩展到「智能体网络」。

### 脚手架经济学：变薄是能力迁移，不是删除

脚手架变薄的实质不是工程量减少，而是流程控制逻辑从脆弱的嵌套判断代码迁移进模型自身的判断力：过去靠「步骤 A 完成才能进 B」的硬编码保证顺序，现在模型能在合理边界内自决步骤。由此出现的「元脚手架」与变薄趋势并不矛盾——协调编排的逻辑被重新集中到更高一层，工程重心从流程控制转向 [[concepts/harness-engineering-framework|Harness 编排]]。对投入定价的含义是：写判断框架的投入会持续贬值，做跨智能体协调与结果验收的投入正在升值。

### 失败模式分类学：三类失配

把对谈中散落的失败信号归拢，可分成三层。技术层失配：层层嵌套的判断框架把模型框死，自动化诉求反而造出脆弱系统。组织层失配：「虚假超级独立感」让人同时启动十个原型各自为战，把产出重新协调到一起的难度超过单点收益，形成蔓延式失控——这与 [[entities/agent-productivity-paradox-collaboration-bottleneck|智能体生产力悖论]] 所刻画的协作瓶颈同构。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:39] 治理层失配：不少团队的安全假设仍停留在二十年前的水平，评估体系缺位又让技术价值无法兑现。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:33] 三类失配的共同根源是同一件事：智能体能力的进化速度持续快于治理、组织与接口等外围层的刷新速度，红利与风险都从这道速度差里产生。

### 组织设计：人员构成不变，杠杆与知识归属改变

工程团队的人员构成半年到一年里几乎没有变化，仍是那批理解系统、知道运维、能值班的人；变化在杠杆——现在几乎每个人都能端到端搭出系统，再指挥自己的 Claude 完成具体工作。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:37] 组织设计的含义是：不必围绕智能体新增头衔与汇报线，而应围绕「放大个体端到端杠杆」重新配置职责与评审点。与之并行的是知识归属的迁移：黑客松项目把老师傅十几二十年攒下的设备判断经验沉淀为可积累的智能体系统，Shopify 的 River 项目则用智能体平台支撑端到端定制化开发——专家知识从「依附于人」转为「沉淀在系统」，人走经验不丢。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:29-31] 优先智能体化的对象，正是这类跟着人走、人走就丢的经验资产。

### 身份层为什么是治理原语

身份与工作流解耦容易被读成单纯的安全特性，但它的战略位置更高：智能体先说明达成结果需要哪些权限，用户授予子集，智能体自开服务账号执行，全程可审计。这套「声明—授权—审计」循环把对智能体的信任从感觉变成可核查的流程，[[concepts/agent-identity-portability|智能体身份可移植性]] 因此成为智能体深度嵌入组织的前置条件。^[raw/articles/anthropic-agent-platform-evolution-three-executives.md:19] 没有这一层，「看不见的操作系统」式愿景——工具隐入基底、智能体主动排查修复——在治理上就不可接受。

## 实践启示

1. **按抽象层位置选平台，而不是只比模型**：优先评估平台替你承接了多少基础设施与脚手架负担、是否提供「结果」级接口；模型代差会被抽象层差距放大或抵消。
2. **现在就跑通「声明—授权—审计」循环**：从服务账号与审计日志起步，把智能体权限当作治理基建来建，而不是等出事故再补。
3. **自主性必须配收敛机制**：放开并行原型之前，先约定同步点与收敛标准，防止「十个原型」式的蔓延失控吃掉协调预算。
4. **ROI 三步走，先点亮速度指标**：按个人→团队→跨团队流程推进，衡量重点放在速度与生产力上，不要一开始就对全部流程做智能化。
5. **扩规模前先还治理欠账**：按智能体时代重写安全检查清单、把评估体系先建起来，否则规模化只会同步放大三类失配。
6. **定期删脚手架，把工程投入挪向编排**：随模型变强主动移除限制性框架部件，把省下的工程精力转投跨智能体协调与结果验收。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

