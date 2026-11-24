---
title: "KnowAct-GUIClaw — 哈工大「Know Deeply, Act Perfectly」自进化 GUI Agent"
created: 2026-08-19
updated: 2026-11-21
type: entity
tags: [gui-agent, openclaw, self-evolution, agent-memory, long-horizon, mobileworld, hitsz, task-orchestration, tool-use]
sources: [raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# KnowAct-GUIClaw — 哈工大「Know Deeply, Act Perfectly」自进化 GUI Agent

## 概述

哈工大张民教授立知大模型团队开源 **KnowAct-GUIClaw**，面向个人助手提出 **「Know Deeply, Act Perfectly」**（知道得越深，行动得越准）范式，在长程 GUI 任务评测基准 MobileWorld 上达到 SOTA。论文 arxiv 2607.12625，代码 github.com/HITsz-TMG/KnowAct。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 核心问题

仅给 OpenClaw 通用智能体「外挂」一个 GUI Agent，仍会遇到：跨应用信息易丢失、操作依赖不断变化的图形界面、执行轨迹用完即弃。许多现有 GUI 智能体每次面对任务都像第一次使用设备——走过的弯路不会成经验，成功操作无法沉淀为可复用能力。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 架构：主智能体编排 + 可插拔 GUI 智能体

框架以具备长期上下文、记忆和工具能力的主智能体负责任务编排，以可插拔的 GUI 智能体负责实时界面操作，在二者间实现「知道得深 → 行动得准」的自进化闭环。它直接回答「只用一句自然语言指令，让个人 AI 助手跨 App 连续干活（提取会议地址→转发联系人→打开地图规划路线）」这一场景。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 深度分析

### Host–GUI 双层解耦：让「不调用 GUI」也成为一种设计收益

KnowAct-GUIClaw 拒绝把所有任务压给一个单体 GUI Agent，而是将长期任务管理与底层界面操作解耦：Host Agent 保留对话、工作区记忆、用户画像、外部工具与全局任务进度，负责判断「做什么」；可插拔的 GUIClaw 负责感知屏幕、规范化动作、连接不同设备后端、校验技能并记录轨迹，决定「如何在界面上完成」。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:44-48]

这种区分让效率成为系统设计的一部分：只有需要读取实时页面、操作登录态应用或进行设备端验证时，Host 才调用 GUI 智能体；若已有上下文、记忆或外部工具足以完成某个信息型子任务，Host 直接处理，减少不必要的 GUI 交互和视觉 Token 开销。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:50-52] 这与 [[concepts/agent-orchestration-patterns|智能体编排模式]] 中的编排者—执行者分工一致：GUI 操作被降级为按需调用的工具路径，而非默认路径。

### Know + Route：上下文汇集与 Blackboard 结构化契约

Know 阶段在真正操作界面前，主动汇集当前对话、用户画像、历史任务、候选应用、相关经验和可用技能；检索到的 UI 操控经验和技能只是辅助信息，不会取代用户当前指令和实时屏幕证据，用户指令模糊时可从记忆中提出明确、可覆盖的默认假设。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:60-62]

Route 阶段把请求划分为单应用任务或有序的跨应用工作流，每个子任务显式声明目标应用、所需输入和应返回的输出；跨应用信息不再依靠一段不断增长的自由文本摘要传递，而是由临时 Blackboard 保存带类型的中间结果——「上游应用观察到什么，下游应用就消费什么」。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:64-66] 必需输入或声明输出缺失时工作流直接停止，而不是让后续步骤猜测或编造，这把「读邮件时间设闹钟」「双购物应用比价」「购物车订单信息发短信」等任务的数据流变成可检查、可恢复的结构化契约。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:68-70]

### Act：统一动作空间、技能复用与 Deep Link「先发现、后验证」

进入 Act 阶段，GUIClaw 在统一的混合动作空间中执行：既可点击、滑动、输入、返回和等待，也可调用已沉淀的技能、经过验证的 Deep Link / Intent 快捷路径，或在需要用户输入和授权时主动请求干预。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:76] 技能不是对旧轨迹的机械重放——每个技能包含适用应用与平台、参数、可靠性统计、操作步骤和状态契约；调用前先检索与当前子任务相关的候选技能，执行中逐步检查当前页面是否满足预期状态，不匹配时可有限恢复、跳过可选障碍步骤或退回普通 UI 操作。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:78-80]

Deep Link 和 Intent 同样遵循「先发现、后验证」原则：应用清单只能证明某个入口可能存在，不能证明它在真实设备上可用，更不能证明它会落到正确页面；系统会实际启动候选入口，结合前台应用、UI 树、截图和参数保留情况进行页面级验证，只有通过验证的入口才进入技能库成为快捷路径。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:82-84] 跨应用比价案例展示了执行层的收益：经过验证的京东搜索快捷路径一步直达商品结果页，淘宝分支需要五次常规 GUI 操作，两个应用观察到的价格经 Blackboard 回传后由 Host 给出比较结果。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:86-88]

### Reflect 与自进化闭环：经验记忆 × 执行技能的双层资产

任务结束后，Reflect 阶段不会简单保存一份冗长日志，而是从轨迹中提炼两类彼此独立、用途不同的长期资产：经验记忆保存可检索的文字策略（成功做法、失败原因、应用选择、界面布局、快捷路径可靠性和恢复建议），用来影响下一次的路由与执行判断；执行技能保存参数化、可运行、带状态验证的操作过程，用来压缩重复导航和高频交互。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:94-98] 系统会过滤空轨迹、过短轨迹、异常终止等缺乏有效信号的记录；对可复用的成功过程，从任务、平台、应用、动作参数、截图和目标控件等证据中抽取新技能；对已调用但执行失败的技能，优先根据失败步骤、页面证据和错误反馈定向修复，而不是无条件生成一个无关的新流程。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:103-105]

这种「记住为什么、学会怎么做」的双层机制让文本经验与可执行技能各司其职，正是 [[concepts/agent-self-improvement-loops|智能体自我改进闭环]] 与 [[concepts/skill-engineering-principles|技能工程]] 在 GUI 场景的落地，也与 [[entities/se-ga-memory-augmented-self-evolution-gui-agents|SE-GA]] 等记忆增强自进化 GUI 智能体同一方向：Mastodon 邀请链接任务中，没有经验记忆时 GUIClaw 会在移动端设置中反复尝试，检索到历史失败经验后，系统能在低层操作开始前修正任务上下文、转向网页管理后台，再由实时页面证据指导后续操作。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:107-109] 量化结果同样支撑这条路线：搭载开源 Kimi-K2.6 在 MobileWorld 上取得 64.1% 成功率（SOTA）；Host 与经验记忆把基础成绩从 55.6% 提到 61.5%，启用自进化技能后达到 64.1%，累计提升 8.5 个百分点；将 Kimi-K2.6 轨迹中提炼的能力迁移至 Qwen3.5-35B-A3B 后成功率提升 16.2 个百分点，技能复用还节省约 6% 的 Token 消耗。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:117-125]

## 实践启示

1. 「外挂一个 GUI Agent」不等于跨应用能力：没有结构化信息传递与可归因经验记忆，长程任务仍会在应用边界处丢失信息、在动态界面前反复试错，走过的弯路不会成为经验。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:16-20]
2. 长程编排的正确分层是把「做什么」与「怎么做界面」分开：信息型子任务交给记忆与外部工具，只有实时页面、登录态应用与设备端验证才动用 GUI 智能体——视觉 Token 是需要节省的成本，不是免费的默认路径。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:50-52]
3. 跨应用数据流应做成显式契约：每个子任务声明目标应用、所需输入与应返回的输出，缺失即停止而不是猜测；失败因此被局限为可继续编排的检查点，而非整条工作流推倒重来。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:64-68,133-137]
4. 技能库的生命力在「先验证后入库」与「失败定向修复」：入口清单不代表可用性，页面级验证通过才能成为快捷路径；对失败技能基于页面证据做修复，比无条件生成新流程更能保住资产质量。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:82-84,105]
5. 经验记忆与执行技能应分开建模：文字策略回答「为什么」，参数化技能回答「怎么做」，两者独立提炼、各司其职，才会同时改善路由判断与执行效率。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:94-98]
6. 自进化的价值应当量化验收：同一开源模型在 MobileWorld 上累计 +8.5 个百分点、跨模型迁移 +16.2 个百分点、技能复用节省约 6% Token——评价自进化系统时，应把「经验与技能的增量收益」与「基础模型强度」解耦报告。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md:117-125]

## 意义

KnowAct-GUIClaw 与 [[entities/mobileforge-annotation-free-gui-agent-kuaishou-zju-2026|MobileForge]]、[[entities/让gui-agent不再边做边忘快手浙大提出memgui-agent攻克长程gui任务|MemGUI]]、[[entities/saas-bench-gui-agent-eval-unipat|SaaS-Bench]] 同属 GUI Agent 前沿，其「主智能体记忆编排 + 自进化」路线与 [[entities/拆解-openclaw-架构一6-阶段流水线与-20-平台的消息归一化|OpenClaw]] 生态深度耦合。

→ [[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz|原文存档]]
