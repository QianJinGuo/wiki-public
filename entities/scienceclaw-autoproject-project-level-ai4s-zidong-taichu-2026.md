---
title: "ScienceClaw AutoProject：紫东太初把 AI4S 从 Task 推向 Project（项目级自主科研引擎）"
created: 2026-08-25
updated: 2026-09-29
type: entity
tags: [ai4s, scienceclaw, autoproject, project2task, taskexecutor, evigraph, zhongke-zidong-taichu, autonomous-research, scientific-agent, multi-agent, project-level-ai, arcbench]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# ScienceClaw AutoProject：紫东太初把 AI4S 从 Task 推向 Project

## 核心命题：AI 在科研中的工作单位，从 Task 走向 Project

AI for Science 正进入新的分水岭。过去 AI 更多作为工具或任务智能体，在科研人员明确目标、边界和路径后完成一个个相对独立的 Task；但真实科研并非任务的简单集合——一个 Project 往往从模糊的研究构想开始，包含多个相互依赖的任务，并随新文献、实验结果和异常数据不断调整研究路线。「把一个个 Task 做好，不等于真正完成一个 Project。」^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

真正的项目级科研，需要 AI 理解科学目标、主动拆解任务、协调任务依赖、持续执行，并根据研究反馈动态调整，最终推动整个项目走向结论。中科紫东太初（中国科学院自动化研究所孵化的多模态大模型企业）旗下科研原生智能体 ScienceClaw 完成关键升级，推出 AutoProject 项目级自主科研引擎，让 AI 的工作单位从 Task 走向 Project——这本质是 AI 科研能力由「工具级」向「系统级」的跨越。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

## AutoProject 的三层核心能力

AutoProject 并非简单延长任务执行链路，而是围绕项目规划、长程执行、证据验证构建项目级自主科研能力，全流程为：项目规划→子 Task 拆解→长程执行→证据验证→动态修复→成果沉淀。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

- **Project2Task（项目级规划）**：回答「这个 Project 应该怎么做？」综合研究目标、文献证据、资源约束与任务依赖，自主确定研究路径、拆解子 Task、规划串并行关系与资产复用，生成可执行可迭代的任务网络。支持横向拆分、纵向拆分、先横后纵、先纵后横四种项目拓扑。从「人定义 Task、AI 执行」到「人提出目标、AI 规划路径」。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

- **TaskExecutor（长程自主执行）**：回答「确定方向后如何持续推进？」打破「一次调用、一次返回」的线性模式，搭建目标驱动的自主科研循环：全局监控 Project 状态、持续捕获实验反馈、自动排查异常、修正研究假设与模型参数；任务失败时可回溯评估、重构任务网络、自主重跑实验。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

- **EviGraph（证据驱动验证）**：回答「结论是否有充分证据支撑？」科研结论不能仅满足行文通顺，每条项目级结论必须依托明确研究问题、科学假设、多组对照实验与真实数据。EviGraph 构建全链路、跨子任务、可核验、可回溯、可修复的可信科研闭环，校验不再局限于单个子任务内部，还兼顾跨任务逻辑一致性；一旦检出偏差，沿证据链定位根因、按需重跑实验。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

## 实证数据（ARCBenchML）

在 ARCBenchML 评测中，EviGraph 综合得分 0.865（最优基线 0.596）；代表结论贴合实验事实的 Result Analysis 指标由 0.442 提升至 0.794；可追溯证据的 Claim Support Rate 达 0.38，较最优基线 0.27 提升 40.74%；实验数据一致性 EDC 为 0.88。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

## 人机协作定位：自主科研 ≠ 无人科研

即便引擎能完成规划、执行、校验、成果沉淀的全链路，科研人员仍发挥不可替代的核心作用——可随时介入，针对 AI 输出的研究路径、实验假设、中间数据开展鉴赏评判、及时纠偏，注入人类领域直觉与科学洞察力。AI 的角色从被动响应需求的科研助手，成长为科研项目的自主推进者。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

## 技术根基：多模态 + 分层自治多智能体架构

ScienceClaw 并非传统「主从 Agent」体系，而是一套面向复杂科研任务的分层自治、多智能体动态协同架构：系统围绕科研目标自动生成任务图，按需调度学科、代码、搜索、数据分析、仿真等专业 Agent，形成可动态组网、协同执行、反馈重规划的科研智能体集群。紫东太初自 2021 年发布全球首个千亿参数三模态大模型，如今紫东太初 4.0 从「理解多模态」走向「多模态推理」。已覆盖生命科学、材料、化学、物理、天文等领域。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

## 深度分析

### Task-vs-Project：真正的自主性分界线

判断一个科研 Agent 的自主性等级，关键不在它能连续跑多久，而在它的「工作单位」是什么。以 Task 为单位的系统（无论单任务内部多复杂）本质上仍把最难的部分——目标定义、路径选择、依赖协调——留给人类；科研项目级自主则要求 AI 从模糊构想出发，自己完成「从目标到任务网络」的翻译。AutoProject 的三层能力恰好对应这次跨越的三个缺口：Project2Task 补的是规划缺口（回答怎么做），TaskExecutor 补的是持续性问题（回答如何推进），EviGraph 补的是可信性问题（回答结论站不站得住）。这解释了为什么单纯拉长 context window 或增加 agent 步数并不能自然产生项目级能力——工作单位不升级，长链路只是更长的线性执行。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

### Project2Task 的四种拓扑与依赖规划

Project2Task 支持横向拆分、纵向拆分、先横后纵、先纵后横四种项目拓扑，这实质上是把软件工程中的任务分解论引入科研规划：横向拆分对应按模块/数据集并行铺开（如带钢缺陷识别中多条研究路线并进），纵向拆分对应沿实验管线逐层深入（如 YOLO 建模的数据处理→建模实验→迭代优化）。更关键的是依赖规划——系统自主识别可并行任务与强制串行环节，并挖掘可跨任务复用的数据与模型资产。串并行判别决定执行效率，资产复用决定边际成本递减，这两点正是把「任务清单」升级为「任务网络」的分水岭：网络中每条边（依赖、复用关系）都是后续执行与验证阶段可追溯、可重构的结构化信息。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

### 长程执行回路与证据验证的咬合

TaskExecutor 与 EviGraph 不是先后两段，而是一个咬合回路：执行阶段全局监控 Project 状态、捕获实验反馈、自动排查异常并修正假设与参数；验证阶段则持续核验「假设是否匹配顶层目标、实验能否支撑假设、各子任务数据能否相互印证、结论是否未超出证据范围」四类跨任务逻辑一致性。一旦验证检出偏差，系统沿证据链定位根因——偏差可能源自上游任务的数据、中间假设本身、或任务网络的结构——再按需重跑实验、修正假设、调整子任务。这种「验证产出的是根因定位而非简单的失败标记」设计，是回路能收敛的关键：没有 EviGraph 的证据链，TaskExecutor 的回溯重跑就只是盲目重试。ARCBenchML 中 Result Analysis 从 0.442 提升到 0.794、EDC 达 0.88，说明这套咬合回路确实抬高了结论与实验事实的贴合度。^[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026.md]

### 与其他自主科研 Agent 的对比

与西方同类系统相比，AutoProject 的差异化在「项目级」这个单位上：Anthropic 的 Claude Science 系列以加速科研工作流（文献、分析、写作）见长，本质仍是把每个环节做成高质量 Task 工具；[[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch]] 用 L0-L4 分级刻画科学发现自主度，其框架同样隐含「最高层级 = 承接完整项目」的判断，与 AutoProject 的实践互为印证。学术侧的评测工作（如 [[entities/sciagentgym-benchmark-multi-step-scientific-tool-use|SciAgentGym]] 的多步科学工具使用基准）仍聚焦子任务级能力，恰好说明项目级评测（ARCBenchML 的跨任务一致性、Claim Support Rate）是新出现且稀缺的度量维度。综述性研究（[[entities/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026|分子科学 LLM Agent 自主性综述]]）也把「从架构设计走向科学自主性」作为演进主线——AutoProject 可视为这条主线在工程产品侧的先行落地。

## 实践启示

1. **评估科研 Agent 时先问工作单位**：不要被「能连续执行 N 步」的宣传迷惑——如果系统需要人类预先定义每个 Task，它仍处工具级；真正的判据是「给它一个模糊目标，它能否产出可辩护的任务网络」。
2. **规划阶段就应显式建模依赖与复用**：Project2Task 对串并行环节和可复用资产的显式识别值得借鉴——任何多任务 agent 工程（不限于科研）中，任务网络的边（依赖、复用）往往比节点（任务本身）更决定系统效率。
3. **验证系统要能定位根因而非只报失败**：EviGraph 的核心启示是证据链设计——验证产出应指明「错在数据、假设还是任务结构」，否则执行侧的重跑退化为盲目重试，长程回路无法收敛。
4. **结论必须被证据范围约束**：Claim Support Rate（0.38 vs 0.27）仍是明显短板，说明「结论不超出证据支撑范围」在 2026 年仍是行业性难题；任何生成研究结论的 agent 都应把可追溯证据率作为一等指标监控。
5. **自主性的正确归宿是「可随时介入」**：AutoProject 把人定位为鉴赏评判者与纠偏者而非操作员，这是比「全自动」更现实的自主性目标——设计长程 agent 时应为人保留低成本的介入点（研究路径、假设、中间数据），而非追求零交互。
6. **把研究成果沉淀为可复用资产**：数据、代码、模型、实验记录与研究方法在项目过程中结构化沉淀（如证据汇总表、产物文件、分析报告），使后续项目的规划阶段可直接复用——资产化是项目级系统能形成复利的前提。

## 相关实体

- [[entities/ai4s-2026-h1-frontier-panorama-yinxi|AI4S 2026 H1 跨学科前沿全景]] — AI4S 赛道全景（尹希），与本文的项目级范式互补
- [[entities/mira-mpa-deep-principle-ai4s-40-sota|深度原理 MIRA + MPA 材料基座模型]] — 另一 AI4S 材料基座技术路线
- [[entities/anthropic推出claude-science-科研界的claude-code|Claude Science 科研 Agent]] — 西方科研 Agent 对比参照
- 多智能体编排
- Agent 规划与推理
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]

→ [[raw/articles/ai4s-project-era-zidong-taichu-scienceclaw-autoproject-2026|原文存档]]
