---
title: "ScienceIDE + Claude 生物分子模型优化：把科学代码库变成智能体可学习的环境"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [agent, rl-environment, science-agent, anthropic, claude, code-optimization, gpu-kernel, benchmark, ai-for-science, sft, scientific-equivalence]
sources: [raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026]
confidence: 0.72
provenance_state: extracted
---

# ScienceIDE + Claude 生物分子模型优化：把科学代码库变成智能体可学习的环境

这条线索由两件几乎同时发生的事情构成：Anthropic 用 Claude 在不到四周内优化了 30 多个开源生物分子模型，证明了「AI 能不能自动化科学代码的性能优化」；紧接着开源的 ScienceIDE 试图回答下一个问题——**怎样让这种能力被持续学习、复用并迁移到更多科研领域**。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## Anthropic：Claude 自主优化 30+ 生物分子模型

Anthropic 发布的这项研究中，被优化的模型包括 AlphaFold3、Boltz-2、Chai-1、ESMFold2、RFdiffusion 等已广泛使用的科学模型。结果显示：在允许少量数值精度变化的情况下，任务**平均加速约 4 倍**；在要求输出完全一致时，**平均加速接近 2 倍**，相关代码同步开源。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

更值得注意的是完成方式：**代码优化主要由 Claude 自己完成**，背后监督它的是两名 Anthropic 技术人员——他们理解生物建模，但**并不具备传统意义上的推理优化或 GPU kernel 工程背景**。这也解释了这批结果的意义边界：它回答的是「AI 能不能做到」，而不是「AI 写出的科学代码离充分发挥硬件性能还有多远」。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## 科学代码的性能空间有多大：PLUTO 案例

AItonomy 团队在等离子体物理与天体物理广泛使用的高性能科学计算代码 PLUTO（代表性论文发表于 2007 年，覆盖流体力学、磁流体力学及相对论情形，支持不同坐标几何、物理模块与数值算法）上做了对照实验，在 M1 Ultra、A100 与 H100 上重新实现其磁流体力学模块。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

在 A100 的一次测试中：AI 生成的第一版实现**已经能够运行**，并在当时检查的算例上得到符合预期的物理结果；但相对团队使用的 128 核 CPU 节点配置，速度提升**还不到 5 倍**。随后团队用一两天时间与 AI 一起分析瓶颈，调整数据布局、内存访问、kernel 组织以及不必要的数据搬运——在不改变所求解物理问题的前提下，这套 GPU 实现的速度**又提高到第一版的接近 3 倍**。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

也就是说，AI 把代码「写出来」之后仍留下数倍性能空间；要继续挖掘，AI 必须能够**测量、诊断、修改、验证、再迭代**——「换个更好的提示词」解决不了数据布局如何匹配线程访问、相邻线程能否合并访存、哪些中间结果值得缓存、kernel fusion 会不会增加寄存器压力并影响 occupancy 这类问题。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## 跑得通 ≠ 算得对：科学等效性作为验收判据

Anthropic 报告中有一个反面例证：为测试优化极限，他们让 Claude 在单个 8 卡 B300 节点上预测 31,000–70,000+ token 的完整病毒衣壳与蛋白区室，推理全部跑通（此前需要多节点集群），但报告写得很直接——**这些结构没有被正确预测**，预测结构发生坍塌，模型在训练上下文近两个数量级之外缺乏泛化能力。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

因此 ScienceIDE 把**「科学等效性」（scientific equivalence）**写成任务是否完成的判据：质量与能量守恒误差是否可控、磁场散度是否满足所选数值方法的要求、激波位置与波传播速度及关键物理量是否与参考结果一致。不先定下这些，程序完全可以靠少算几步、降低分辨率或跳过重要过程变快，同时失去原来的用途。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## ScienceIDE：把科学代码库变成可执行、可验证的学习环境

ScienceIDE 由硅谷非营利组织 AItonomy Foundation 于 2026 年 9 月 17 日发布（赞助方为 PhAI Labs 与阿里 Qwen），做法是：由领域专家定义科学算例与验收标准，再与 AI 一起把代码库整理成**可执行环境**；在这些环境里 AI 完成任务、运行程序、检查科学结果，而**经过验证的交互经验被用于评测、监督微调和强化学习**。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

论文报告的集合包含来自 **27 个科学代码库的 64 个环境、2,812 个任务与 1,076 项可执行科学检查**，覆盖天体磁流体、空间等离子体、海洋气候、引力 N 体、光子学与电磁、材料、量子、相对论、粒子探测器、免疫学等方向；任务分七类：加速、修复、发现、复现、集成、标定、实现。需要说明的是，当前任务主要集中在代码修复与功能实现，**加速类任务目前还只有初步实例**。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## ScienceIDE-Hard：最强模型也只做到 67%

团队从中挑出 85 个难任务构成公开评测集 ScienceIDE-Hard（来自 PLUTO、Athena++、MITgcm、LAPS、PHANTOM 等环境，含 52 个修复任务与 33 个实现任务），判定标准是在任务验收条件下**与私有科学参考答案一致，而不是执行成功**。参评的是来自 8 家厂商的 15 个模型，通过 Codex、Claude Code 或 Gemini CLI 执行，每个 episode 一小时预算。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

结果：最强的 **Claude Fable 5.1 做到 67.1%**，Claude Opus 5 为 64.6%，GPT-6-astra 为 63.1%，而**一半以上的前沿模型停在 30% 以下**。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## 科学不只是应用场景，也可能是训练场

团队用经过验证的科学交互轨迹做监督微调，训练并开源了 **PhAI-IDE-4B、9B、72B** 三个规模的模型，结果显示提升的不只是科学任务本身，代码、推理、知识三类通用基准同时受益：72B 上 APPS Introductory 从 0.609 提升到 0.672、LiveCodeBench Execution 从 0.539 提升到 0.594；9B 上 BBH Word Sorting 从 0.240 提升到 0.576。这与常规代码任务相比，科学任务天然更长程、更依赖对物理约束与数值行为的理解。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

团队公开表示希望把 Anthropic 此次开源的 36 个优化包整理成 ScienceIDE 环境——这些包附带参考实现与可度量的验收标准，条件接近理想，如果成立，社区将不只是使用这批优化结果，而是能在这类工作上训练和评测模型。目前已参与的研究者来自伯克利、CMU、康奈尔、加州理工、哈佛、MIT、牛津、UCL、普林斯顿、斯坦福等二十余所机构，以个人身份参与。^[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026.md]

## 资源

- ScienceIDE 技术报告：ScienceIDE — Turning World's Scientific Codebase into Agent Learnable Environments（arXiv:2609.19134，<https://github.com/aitofound/ScienceIDE>，模型系列 <https://huggingface.co/collections/AItonomy/scienceide-model-series>，项目页 <https://aitonomy.org/projects/scienceide>）
- Anthropic 原文：<https://www.anthropic.com/research/claude-uplifts-biomolecular-modeling>

## 相关

- [[entities/anthropic-biology-agent-data-infrastructure-virbench|Anthropic：生物学 Agent 的瓶颈在数据基础设施]]
- [[entities/anthropic-com-research-making-claude-a-chemist|Making Claude a chemist]]
- [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]]
- [[concepts/agent-self-improvement-loops|Agent 自我改进循环]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[entities/agent-assisted-sglang-development-lmsys-2026-07|Agent 辅助 SGLang 开发]]
- [[entities/geora-geometry-aware-lora-rlvr-meituan-2026|GeoRA：面向 RLVR 的低秩适配]]
- [[entities/rlvr-entropy-collapse-steer-acl-2026-outstanding|RLVR 熵坍塌与 STEER]]
- [[moc/layer-3-agent-engineering|MOC：Agent 工程（Layer 3）]]

→ [[raw/articles/scienceide-claude-biomolecular-code-optimization-agent-environments-2026|原文存档]]
