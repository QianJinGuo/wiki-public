---
title: "Macaron-V1：基于 GLM-5.2 的混合 LoRA 个人智能体模型"
created: 2026-07-22
updated: 2026-11-20
type: entity
tags: [model, open-source, llm, chinese-ai, lora, agent, mind-lab, glm-5.2]
sources: [raw/articles/glm-5-2-mixed-lora-200m-context]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Macaron-V1：基于 GLM-5.2 的混合 LoRA 个人智能体模型

## 深度分析

Macaron-V1 是心洲科技旗下前沿实验室 Mind Lab 推出的个人智能体模型系列，包含 **Venti（748B）** 和 **Tall（35B）** 两个版本。Venti 基于 [[entities/z-glm-5.2|GLM-5.2]] 基座，采用 **MoL（Mixture of LoRA）** 架构，将基座模型冻住，挂载四个 1B 规模的 LoRA 专家模块，分别负责对话（L0 Chat）、智能体（L1 Agent）、代码（L2 Coding）和界面渲染（L3 UI/A2UI）。这种插件式架构天然支持持续学习，新增能力只需训练新 LoRA 接入即可，不会动摇已有知识体系。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md]

Venti 原生支持 **2M 超长上下文**，这得益于团队自研的 **LongStraw** 训练技术——共享长提示词只计算一次，存为可复用常驻状态，只重放有学习信号的回复分支。官方数据表明，8 张 H20 上可将 Qwen3.6-27B 的 GRPO 训练推到 210 万 token，32 张 H20 上训练 GLM-5.2（78 层 MLA/DSA、256 专家 MoE）全模型同样达到 210 万 token。这一能力使模型能处理极长的智能体交互轨迹。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md]

在训练基础设施方面，Macaron-V1 使用 **MinT** 管理底层训练，Actor 与 Learner 之间只传递 Adapter，配合百万级 Adapter 目录可支撑万亿参数模型，与 [[entities/mind-lab-lora-continual-learning-system|Mind Lab 的 LoRA 持续学习体系]] 深度契合。其上运行的 **HyperRSI** 实现递归自我改进闭环：从种子任务生成更难的任务，筛掉低价值样本，对生成轨迹做审计，反复迭代直到收益收敛，再将经验固化为新参数。训练框架 **MindForge** 将 Pi 和 Claude Code 生产环境的 Harness 直接接入强化学习，通过 **HCP 协议** 对齐训练与线上环境的任务信息，保证练用一致。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md]

评测方面，团队自建 **ChatBench**（评估长智能体上下文中的诚实度与边界意识）和 **LivingBench**（跨数周持续互动的纵向评测，含动态噪声与动态用户机制）。同时在 VitaBench、PinchBench（个人生活智能体）、SWE-Verified、DeepSWE、TerminalBench 2.1（编程与终端）和 UI4A-Bench（生成式 UI）等通用基准上与 Opus 4.8、GPT 5.5 等前沿模型对比。官方结论显示，在个人生活和生成式 UI 两条轴线上，Venti 能打平甚至超越前沿基线，编程能力保持在接近前沿的水平，同时保留开源可自托管的优势。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md]

实测中，UI4A 生成式 UI 能力令人印象深刻：输入口语化财务描述后，模型自动提取多笔交易数据，渲染出包含环形图、资产走势折线图、可编辑数据网格的交互式财务控制台，且数据修改后图表能联动更新。Coding 方面，L2 专家成功生成了复杂的 Three.js 机械太阳系仪和 WebGL 水墨山水长卷（含 12 段落、古琴散板音乐合成、严格状态机管理的诗句浮现）。Agent 方面，模型自主探索 FastAPI 仓库，定位了一个长期未解决的多 Query 参数模型 issue，给出修复方案并通过了三千多个现有测试。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md]

### 双版本矩阵：Venti 与 Tall 的基座分化

Venti 为 748B 参数（744B 基础模型加 4 个 1B LoRA），官方称其为全球首个基于 GLM-5.2 完成后训练的模型；Tall 仅 35B，基于通义千问 3.6 后训练而来，专为本地部署设计，面向对隐私与成本敏感的场景。同一产品线在旗舰能力与可部署性之间做了显式的基座分化，而非单模型缩放。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:31-37]

### 四大专家的分工边界与 Preview 版演进

四个专家的分工并非泛泛的领域划分：L0 Chat 是对话与指令遵循主干；L1 Agent 面向个人生活场景的重度工具使用，覆盖拖得较长、变数较多的复杂任务，且 V1 已把 Preview 版本 OpenClaw 相关的 LoRA 合并进 L1；L2 Coding 管代码理解、SWE 任务与终端操作；L3 UI/A2UI 负责 UI4A 渲染和 UI 驱动的操作。作者将其比作给知识渊博的底座配四个独立外脑——不同来源的技能在各自适配空间里优化、互不打架，这正是「治愈技能冲突」的由来。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:41-51]

### LongStraw：瓶颈在训练端而非推理端

原文澄清了一个容易误读的点：推理端读百万级 token 不难，真正的瓶颈在强化学习训练端——同一提示词要评估多份回复、保留激活、累积梯度，固定硬件上很容易卡在二十多万 token。LongStraw 把共享长提示词只计算一次、存成可复用的常驻状态，后续只重放真正有学习信号的回复分支；在此之上，8 张 H20 的常驻前缀训练最高能推到 446 万 token，比常规路径的 210 万更高一档。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:475]

### 实测工作流：端到端自我验证而非一次性生成

三轮实测的共同特征是模型在 Claude Code 中边构建边调用浏览器截图、实时测试渲染效果并自行定位故障：Coding case1（Three.js 机械太阳系仪）构建过程中持续截图自检；case2（WebGL 水墨山水长卷）耗时约半小时，十二段落的滚动阻尼、视差速度与墨色浓淡过渡被评价为「很有呼吸感」。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:224] ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:352] ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:448-450] Agent 任务耗时约 2 小时：模型先确认 issue 在最新代码上依然成立、刻意避开维护者已声明要私下处理的伪需求，定位出单模型参数在文档生成与运行时解析两处的判断缺陷后抽公共辅助函数修复，新增六个测试与教程八例，原有三千多个测试全部通过，并用回退代码证明唯一失败项是环境问题。作者核实其对应 PR #12481 真实存在且自 2024 年 10 月起未合并。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:459-467]

## 实践启示

1. **插件式扩展优于重训**：新增能力只需训练一个新 LoRA 接入，不动基座已有知识体系——私有模型的能力演进可直接借鉴「冻结基座 + 挂载专家」模式。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:51]
2. **长上下文先分清两端瓶颈**：推理能读百万 token 不代表训练能训百万 token；自建 RL 管线遇长上下文时，共享前缀常驻化（LongStraw 思路）比堆卡更划算。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:475]
3. **个人智能体评测必须纵向化**：ChatBench 盯长上下文中的诚实度，LivingBench 用动态噪声、动态生存环境、动态用户三机制做跨数周纵向评测——静态跑分测不出计划偏离后的重规划能力。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:483]
4. **Agent 自主工程任务的验收清单**：确认需求在最新代码仍成立、避开已有人认领的伪需求、跑全量测试、用回退代码区分环境性失败与改动引入的失败——这套流程可直接搬进代码智能体的评测设计。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:459-467]
5. **生成式 UI 重构轻工具获取方式**：口语化流水一句话渲染成可编辑、图表联动的财务控制台，意味着健康打卡、看板类小工具可以「当场捏出来」，不必再下载 App。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:192-194]
6. **按能力轴线选型而非唯跑分论**：官方明确编程线只求「接近前沿」、不硬称最强，个人生活与生成式 UI 两线打平或超越前沿基线——开源可自托管是另一条独立的价值轴。 ^[raw/articles/glm-5-2-mixed-lora-200m-context.md:485]

## 相关实体

- [[entities/macaron-v1-lora-moe-architecture|Macaron-V1 的 LoRA-MoE 架构]]
- [[entities/mind-lab-lora-continual-learning-system|Mind Lab 的 LoRA 持续学习体系]]
- [[entities/z-glm-5.2|GLM-5.2]]
- [[concepts/agent-self-improvement-loops|智能体自我改进闭环]]
- [[concepts/agent-evaluation-benchmark-frameworks|智能体评测基准框架]]

→ [[raw/articles/glm-5-2-mixed-lora-200m-context|原文存档]]
