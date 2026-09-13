---
title: "MobileForge：无标注手机 GUI Agent 适配系统（快手、浙大）"
created: 2026-07-09
updated: 2026-09-14
type: entity
tags: [gui-agent, mobile-agent, annotation-free, reinforcement-learning, grpo, kuaishou, zhejiang-university]
sources: [raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# MobileForge：无标注手机 GUI Agent 适配系统（快手、浙大）

MobileForge 是浙江大学 APRIL 实验室、快手主站技术部和清华大学联合提出的手机 GUI Agent 无标注适配系统，由一个闭环数据飞轮组成：真实 App 探索 → 自动任务生成 → 分层评估 → 策略优化。论文题为 *MobileForge: Annotation-Free Adaptation for Mobile GUI Agents with Hierarchical Feedback-Guided Policy Optimization*，全链路已开源。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

## 核心问题

手机 GUI Agent 进入真实应用场景面临三大瓶颈：

- **碎片化**：不同 App 页面结构、功能入口、交互逻辑差异大，版本更新频繁导致界面变化
- **长链路**：移动端任务可能跨多个 App 传递信息，稀疏奖励信号无法定位失败原因
- **数据依赖**：有监督 GUI 学习方法需要人工编写任务、录制专家轨迹、标注奖励信号，成本失控

与团队之前的 MemGUI Agent（专注长程记忆）不同，MobileForge 解决的是从零开始的领域适配问题——不依赖任何人工标注或预定义任务库。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

## 系统架构

MobileForge 由两个耦合组件构成：^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

### MobileGym：交互与评估底座

MobileGym 解决数据来源问题，包含三个关键阶段：

1. **目标 App 探索**：结合 APK 声明的 activity 结构和当前截图，生成面向功能的探索目标。探索过程采用深度优先遍历，轨迹被记录为操作前后截图、执行动作、目标元素、元数据和自然语言摘要，形成证据池
2. **MobileGym-Curriculum**：将探索证据转化为可执行任务。每个任务表示为五元组：任务指令、预估步数预算、核心功能、变化类型和前置条件
3. **MobileGym-Critic**：不训练奖励模型，而是用 agentic hierarchical evaluator 对完整 rollout 进行分层评估——输出轨迹级 outcome label、步骤级 process label 和纠错 hint

### HiFPO：层级反馈引导的策略优化

HiFPO 将失败经验转化为训练信号，包含四条关键设计：^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

1. **带提示的多次尝试**：对每个任务连续尝试 K 次，前一次失败产生的纠错 hint 追加到下次任务指令
2. **任务过滤**：计算经验成功率 SR(x)，全成功任务被移除（已掌握），全失败和部分成功任务保留（有学习价值）
3. **轨迹与步骤选择**：从保留任务中选择高质量轨迹，只保留被 Critic 判定为合理的局部步骤，将长链路轨迹拆为密集 step-level 训练样本
4. **Hint-contextualized step-level GRPO**：每个 step-level 样本包含任务、截图、交互历史和纠错提示，模型在带 hint 的状态下采样多个候选动作，用规则化 GUI action reward 进行组内比较

## 关键实验数据

| 模型 | 设置 | 指标 | 分数 |
|------|------|------|------|
| Qwen3-VL-8B | 基线 | AndroidWorld Pass@3 | 55.2% |
| ForgeQwen3-8B (900 tasks) | 无标注适配 | AndroidWorld Pass@3 | **67.2%** |
| GUI-Owl-1.5-8B | 闭源数据专用基线 | AndroidWorld Pass@3 | 69.0% |
| **ForgeOwl-8B** | 无标注适配 | AndroidWorld Pass@3 | **77.6%** |
| ForgeOwl-8B | 跨域泛化（无 MobileWorld 训练数据） | MobileWorld GUI-only | **41.0%** |

实验共生成 3249 个 AndroidWorld 候选任务（20 个 App、527 个源轨迹），且展示了清晰的数据规模扩展趋势。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

## 深度分析

### 一、瓶颈不在"点得准"，而在"适应不了"

移动端困难不在"会不会点"：App 生态碎片化、界面随版本变动，任务常跨多页面，稀疏奖励无法定位失败在哪一步。已有无标注方法还差统一底座，策略优化亦把 rollout 当孤立样本。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

### 二、用分层评估器替代人工标注

MobileGym-Critic 不训奖励模型（无标注下没有奖励标签可训），而是用 agentic hierarchical evaluator 分层判定 rollout，输出轨迹级 outcome label、步骤级 process label 与纠错 hint，把“任务是否完成”与“每步是否合理”拆成两层信号。它解决长链路信用分配：失败轨迹常含正确动作，成功轨迹也可能有冗余动作，二值标签会把信号丢掉。

Critic 消融显示换成 Qwen3-VL-8B 当决策模型，仍能把 Pass@1 从 40.5% 抬到 44.8%、Pass@3 从 55.2% 抬到 60.3%：闭环不绑定特定闭源评价器，而评价器质量即数据质量上限。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

### 三、自博弈闭环：累积经验 + 步骤级筛选

HiFPO 前半段是带提示的多次尝试：同一任务连续尝试 K 次，失败后由 Critic 生成的 hint 追加进下一次指令。收益来自经验累积而非采样量：不加 hint 时总成功率 52.0%，加 hint 后升到 77.0%，Pass@3 从 49.0% 到 72.5%。

后半段是过滤与选择：按经验成功率 SR(x) 去掉"全成功"的已掌握任务，保留全失败与部分成功任务（SR ∈ [0.0, 0.9]）；优先选步骤质量最高的成功轨迹（全失败时取局部合理步骤占比最高者），只保留 Critic 判定合理的步骤。长轨迹因此切成密集的 step-level 样本且不强化错误动作。^[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge.md]

### 四、基准数字与泛化边界

域内：Qwen3-VL-8B 在 AndroidWorld 上 Pass@3 为 55.2%，用 900 个自动任务适配后 ForgeQwen3-8B 达 67.2%，接近闭源数据训练的 GUI-Owl-1.5-8B 的 69.0%；继续适配得到 ForgeOwl-8B 的 77.6% Pass@3。域外更值得警惕：完全不用 MobileWorld 数据的 GUI-only 任务上，ForgeOwl-8B 只从 37.6% 提到 41.0%，ForgeQwen3-8B 从 7.6% 到 10.3%——适配只能强化基座已有的手机 GUI 能力。失败热力图也划出边界：提升集中在 verification、search、complex UI、screen reading 等界面锚定能力，而 game-playing、multi-app、memorization 仍很难。

## 实践启示

1. **先选基座再谈适配**：域外 41.0% 对 7.6% 说明收益是基座能力的倍数项。
2. **让失败在任务内累积**：K 次尝试 + hint 注入把尝试串成经验链，比加大采样量有效。
3. **过滤已掌握任务而非失败任务**：全成功任务没有梯度，关键是步骤级选择救回正确动作。
4. **任务生成锚定探索轨迹**：只基于 landing screen 会让课程退化为首页功能。

## 与相关实体的关系

- 同团队先前的 [[entities/让gui-agent不再边做边忘快手浙大提出memgui-agent攻克长程gui任务.md|MemGUI Agent]] 解决长程记忆问题，MobileForge 解决的是从零开始的无标注适配——两者互补而非竞争
- 与 [[entities/saas-bench-gui-agent-eval-unipat.md|SaaS-Bench]] 和 [[entities/se-ga-memory-augmented-self-evolution-gui-agents.md|SE-GA]] 等同属 GUI Agent 评估与训练方法体系

→ [[raw/articles/不用人工标注gui-agent跑起数据飞轮快手浙大开源mobileforge|原文存档]]
