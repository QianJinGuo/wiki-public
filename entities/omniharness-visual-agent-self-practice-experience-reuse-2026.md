---
title: "OmniHarness：让视觉智能体自主练习并把验证过的流程沉淀成可复用策略"
created: 2026-09-21
updated: 2026-09-21
type: entity
tags: [agent, harness, experience-memory, self-improvement, multimodal, benchmark]
sources: [raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026]
confidence: 0.7
---

# OmniHarness：让视觉智能体自主练习并把验证过的流程沉淀成可复用策略

## 核心要点
- 论文：OmniHarness（arXiv **2609.16057**，北京航空航天大学 × 香港中文大学 × 新加坡国立大学；项目主页 omniharness.github.io，代码 github.com/OmniHarness/OmniHarness）。核心命题是「AI 完成一次任务之后还能留下什么」——让视觉智能体主动练习、检查结果，再把验证有效的流程留下来。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- 结果：ComfyBench 创意任务解决率 **95%**，超过表中最佳对照 SymbOmni **27.5 个百分点**；Codex + GPT-4o 配置下总体解决率 **92.5%**。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- 机制三步：**选在值得探索的地方练习** → **让检查跟上每次执行** → **让成败都成为下一次的依据**（成功抽象为策略入库，失败记录证据/原因/修正）。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- 迁移性：把自主练习后的**策略库冻结**，再适配到其他智能体的知识接口（宿主保留原控制流程、模型不微调），三个系统的总体解决率都提高；只练过图像任务的策略库还能用于视频工作流。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- 边界：复杂任务 83.3% 仍未超过 ComfyMind 的 85%；自主练习、多轮验证和修复都增加计算开销，检索效率与推理预算仍有优化空间。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

## 深度分析

**命题：把「经验」从模型参数里拆出来，放到可调用的策略库里。** OmniHarness 给出的路径是让视觉智能体主动练习、检查结果，再将有效流程留下，从而在面对下一个需求时手里多一份经过验证的方法；论文的落点是「模型参数不变，可调用的方法随实践更新」——经验外置于模型，而不是靠继续训练内化。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

**评测口径把「能跑通」和「能交付」分开。** 表中 Pass 看流程能否执行，Resolve 看输出能否满足任务要求；OmniHarness 两种配置的 Pass 均为 100%（ComfyMind 与 SymbOmni 也已做到），因此区分度全部落在「可运行的流程交出了多少合格结果」上。同以 GPT-4o 为推理骨干时，OmniHarness 总体 89.5% vs ComfyMind 83%，创意任务上两者分别为 95% 与 57.5%；换用 Codex 规划后总体由 89.5% 升至 92.5%、复杂任务由 76.7% 升至 83.3%——不同规划配置对应不同的多步任务表现，规划骨干本身也是变量。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

**第一步「练习选题」：新颖性 × 能力边界，偏向有基础但仍有挑战的任务。** 自主练习发生在下游任务明确之前；只做熟悉的题难以补齐空缺，挑战太远又缺少方法支撑。系统每轮默认生成 10 道候选题，以新颖性和能力边界评分的乘积决定练哪一道；新颖性用「同类情境练得越多、得分越低」的方式度量（图生图以源图区分情境，文生图共享一个标记），能力估计按所需能力取平均以避免「能力项列得多就占优」；每项能力的支持来自该能力下适用且未停用流程的最高可靠性，可靠性用 **95% Wilson 置信区间下界** 估计，整道题取各能力估计的最低值——即先看短板，一处缺少可靠方法仍会拖累整体。随后把能力估计转成选题分数：中间值处得分最高，估计值趋近 0 或 1 都会下降。这套评分是选题启发式，用于表达能力对任务的支持程度。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

**第二步「检查」：确定性验证 + 局部修复，保留已验证部分。** 系统安排步骤和依赖，把代码编译成 ComfyUI 工作流，再检查流程能否运行、各步效果及最终目标是否达成。以花田任务为例：假如红花已偏离要求，下一步继续沿用偏差就可能进入新画面——所以失败后系统定位问题并做局部修复，尽量保留已验证部分，必要时请子智能体协助，并在重试预算内再次检查。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

**第三步「沉淀」：成功是模板，失败也是资产。** 验证成功的流程被抽象入库，失败则记录证据、原因与修正方法；等价流程会合并，可靠性随调用更新，经验继续影响后续选题和执行，策略本身还要接受新的成败记录检验（影响可靠性与调用优先级）。具体到策略的形态：保存的策略包含**工作流模板、适用条件和资源依赖**，当前图片、具体描述等实例输入则被抽离；后续调用先检查是否适用，再绑定新输入，按要求调整步骤或组合多条策略。这种「模板 + 适用条件 + 资源依赖」的抽象，与技能库/知识库这类可复用能力载体的设计取向一致。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

**迁移实验：冻结策略库 + 换宿主，模型不微调也能涨点。** 团队把自主练习后的策略库冻结，再适配到其他智能体的知识接口，宿主保留原有控制流程、模型也不微调，交付内容包括成功流程模板与失败证据及修正办法，由其他智能体通过自身检索、规划、执行机制使用：ComfyAgent 由 32.5% 升至 57.0%，ComfyMind 从 83.0% 提高到 88.0%，SymbOmni 由 86.0% 达到 89.0%，对应创意任务增幅依次为 27.5、17.5、10 个百分点，且创意任务提升均超过各自总体增幅。还有一层跨模态迁移：只练过图像任务的冻结策略库被用于视频工作流，视频总体解决率 85.9%（高于表中最佳对照 5.1 个百分点）、视频到视频 80.0%（高出 13.3 个百分点），其中图像策略负责内容构建、参考保持等操作，时序处理仍交给视频专用组件，二者需经适配与组合。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

## 实践启示

- **经验应当是可检验的资产，而不是对话历史。** 该框架把「成功流程」抽象为带适用条件与资源依赖的策略，把「失败」记录为证据 + 原因 + 修正方法，并让可靠性随每次调用更新——这比把历史轨迹塞回上下文更接近「可复用能力」的定义。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- **练习要按短板选，而不是按兴趣选。** 整题能力估计取各能力项的最低值 + 95% Wilson 置信区间下界，等于系统性地把练习预算投向「最不可靠的环节」，而不是已经熟练的环节；得分函数在中间值最高，避免在过易或过难的任务上浪费算力。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- **验证的粒度决定经验能否被复用。** Pass（能否执行）与 Resolve（是否满足要求）分离，是本框架能区分「流程可用」与「结果合格」的关键；只记录「跑通了」会把大量不合格输出当成成功经验入库。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]
- **策略库与执行体解耦，迁移才成立。** 冻结策略库、模型不微调、宿主保留控制流程的条件下仍能提升三个外部系统，说明收益来自「经验接口」而非模型改动；代价是自主练习、多轮验证与修复带来的额外计算开销。^[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026.md]

## 相关实体
- [[concepts/agent-self-improvement-loops]]
- [[concepts/agent-harness-engineering-paradigm]]
- [[concepts/agent-memory-systematic-framework]]
- [[entities/evoscientist-experience-memory-autoskills-2026]]
- [[entities/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl]]
- [[entities/agent-harness-engineering-survey-2026]]

→ [[raw/articles/omniharness-visual-agent-self-practice-experience-reuse-2026|原文存档]]
