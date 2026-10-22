---
title: "Scinetics Chain of Science Tokens 与 Chain of Structures：把酶设计变成可验证的结构推理链"
created: 2026-10-19
updated: 2026-10-22
type: entity
tags: [ai4s, science-agent, protein-design, reasoning, enzyme, self-improvement]
sources: [raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026]
confidence: 0.65
provenance_state: extracted
---

# Scinetics Chain of Science Tokens 与 Chain of Structures：把酶设计变成可验证的结构推理链

Scinetics 团队提出「科学 Token 链（Chain of Science Tokens）」：把序列、结构、图像、组学和测量数据表示成模型可读写的科学信息单元，与语言共同构成推理过程——文字提出目标，科学状态承载中间决策，每一步的产物成为下一步的依据。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 一次「思考」对应一次结构变化

该方向的核心主张是让 AI 直接用科学对象思考，而不是先转成文字再推理。以蛋白质结合口袋设计为例：模型读取局部结构、尝试放入一种侧链、检查空间冲突再调整方案；推理不再是纯文本独白，而是一串可被后续计算和实验检验的结构状态。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## Chain of Structures：酶设计的四轮结构链

团队在 Chain of Structures（COS）中选择酶设计作为突破口，把科学 Token 链落实为四轮逐步成形的蛋白质结构：第一轮确定催化核心（按反应需求选择承担关键化学作用的残基、构建局部几何）；第二轮完善活性位点（补充底物识别与支持反应的相互作用）；第三轮构建全局骨架（让蛋白质包围并支撑活性位点，同时保留底物进入空间）；第四轮补全序列与侧链，把局部设计拓展为完整蛋白质。每一轮都产生实际结构状态，后续设计必须尊重已形成的约束。工作由 MIT、哈佛大学、剑桥大学、香港科技大学与 Scinetics 等机构研究人员共同完成，张载熙为通讯作者。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

连接推理与生成是关键：COS 一方面传递明确的残基与位置约束，另一方面利用连续表示把结构训练信号传回推理端，使模型学到的不只是「怎样描述方案」，还有「怎样提出生成器能够实现的方案」。当前四轮流程向前单向推进，更长的推理、回退与纠错仍是待探索方向。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 结果与消融

在酶设计基准对比中 COS 报告六项指标五项排名第一（覆盖功能匹配、结构置信度与结构自洽性）：三级酶分类一致率 EC3 达 57.0%（超过 CodeFP 的 55.3%），pLDDT 79.3，TM-score 0.729。消融显示结构链的价值来自推理与生成的配合：去掉推理链 EC3 降至 53.4%；保留推理链但去掉连续接口则降至 51.8%——只有两者结合才取得最佳表现。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

需要强调的边界：这些结果是计算层面的支持（中间结构可成为设计过程的一部分并影响最终产物），真实酶活性、选择性与稳定性仍待湿实验检验。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## ScienceRSI：从单次推理到跨代学习循环

配套 Perspective 提出 ScienceRSI（科学递归自我改进）：模型与智能体产生候选科学 Token 链，模拟器、数据库、实验与领域专家负责筛选；有证据支持的轨迹进入训练集，失败记录也被保留用于训练更强的下一代模型，新模型再产生新的候选与任务经验，继续推动循环。科学任务提供了具体检验对象——结构是否冲突、模拟是否满足约束、实验能否重复；模型提出候选，外部证据决定哪些经验值得学习，并明确区分预测、模拟与实测。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

远期目标被表述为科学超级智能（Scientific Super Intelligence）：同一个固定模型在专业科学任务上超过垂直模型、在广泛科学推理上超过同规模通用语言模型；Perspective 给出 1000 亿至 3000 亿参数、10 万亿至 30 万亿训练 Token 的探索窗口作为可检验假设。COS 是这一长期方向的早期案例，而非最终模型。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 深度分析

### 可验证性：奖励信号从「评审」换成「打分器」

把一次推理绑定到一次具体的结构突变，最直接的后果不是模型变聪明，而是**验证成本塌了下来**。自由形式思维链的产物是文本，判断对错只能靠人读或靠另一个模型当裁判；后者的代价是奖励可被措辞取悦，模型学会的是「写得像对的」。COS 把每轮推理的落点规定为可计算的蛋白质状态，检查项随之变成一组物理与几何量：残基位置是否合理、侧链是否空间冲突、pLDDT 是否站得住、结构是否自洽。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

关键在于这些检查**局部、廉价、可批量**——不必等湿实验，甚至不必调用第二个大模型；奖励黑客的空间被物理约束压窄，话术无法让一个原子重叠的侧链变合法。「残基与位置约束」管约束，「连续表示回传训练信号」管梯度，两个接口缺一不可。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

但打分器只保证「像真的」，不保证「能用」：几何自洽的酶照样可能没有活性。

### 四轮结构链：受约束的探索-利用调度

四轮划分看似任务分解，实质是一条**约束逐步收紧的搜索路径**：催化核心 → 活性位点 → 全局骨架 → 序列与侧链。顺序不能倒置，因为每轮都在为下一轮缩小可行域——先定催化残基，活性位点的相互作用才有锚点；先定骨架，侧链补全才不会堵死底物通道。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

一次性生成整条蛋白，要求模型在巨大空间里同时满足互相牵制的约束；结构链把它拆成四次「小范围探索 + 立即固化」，代价是失去全局重排的自由，收益是后续搜索被前一步的既成事实剪枝。

四轮是**观察到的平台**而非原理上限。加轮数的收益来自更细的化学微调，但每多一轮就多一次不可回退的决策，错误沿链累积；减轮数则前期决策承担过重的全局责任。团队明说流程单向推进、回退未实现——**没有回退的结构链，是一条不能回溯的深度优先搜索**，第二轮走偏，后三轮都在为错误的核心做装修。四轮够用，因为它在「粒度细到每步可控」与「链条短到错误不致命」之间踩到了甜点。

### 三组消融各读出什么

去掉推理链，EC3 从 57.0% 降到 53.4%：文字层推理本身仍有贡献，但约 3.6 个百分点，并不惊人。保留推理链、去掉连续接口，则降到 51.8%——比完全没有推理链还差。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

最后这个数字最值得琢磨。它意味着**「会描述方案」与「能提出生成器可实现的方案」是两种能力，前者单独存在时有害**：模型在文本空间规划出一条自洽路线，却无法翻译成生成器认得的连续结构条件，中间产物就从约束变成噪音。这与 [[concepts/verifier-driven-development|验证驱动开发]] 同构：没有反馈通道的规划，只是把幻觉写得更长。

「更多/更少轮数」与「用奖励模型替代结构打分」两组对照，报道未给数字，此处不代为编造；可预期的是奖励模型会重新引入代理目标，把奖励黑客从「骗人」变成「骗打分模型」。

### ScienceRSI：跨代蒸馏与自我训练的回音室

循环设计有个易被忽略的细节：**失败轨迹也被保留**。只学好链，模型只见得到可行域内部，学不到边界在哪；负样本提供的「哪里会撞墙」对搜索类任务更值钱。外部证据承担筛选，模型只提候选——改进方向由实验事实而非模型自评决定，这是它区别于 [[concepts/ai-self-improvement-bootstrapping|自我改进引导]] 之处。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

但自训练的风险同样清晰。一是**分布收窄**：用自身采样反复重加权训练集，几代后多样性塌陷，模型越来越擅长已经会的那类设计。二是**对打分器过拟合**：打分器成为主筛选器后，模型朝其偏好漂移，pLDDT 上涨而活性不动。三是**验证吞吐成为天花板**：循环转速由能验证多少条链决定，它更像被实验带宽卡住的流水线。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

### 诚实的边界

其一，**评测面窄**：单一基准、六项指标、有限几个方法；EC3 只领先 1.7 个百分点，属于易被种子或调参抹平的量级。其二，**证据是计算而非实验**：活性、选择性、稳定性全部待湿实验。其三，**跨域迁移未验证**：分子、材料、结合剂虽同属「科学对象可状态化」，但约束硬度差异极大——酶设计成为突破口，正因物理约束最硬、打分工具最成熟；换领域，这根支柱会立刻变软。其四，**复现门槛不低**：千亿级参数假设之外，还需结构生成器、打分栈与领域数据管线。

## 实践启示

1. **先问「这一步能被什么自动检查」，再决定让模型多想一步。** 落不到可计算检查项上的推理，长思维链只带来更长的幻觉。
2. **把复杂生成改写成有顺序的约束固化序列。** 找出那些「一旦定下就能剪枝后续搜索」的决策并前置；催化核心 → 活性位点 → 骨架 → 侧链就是样板。
3. **专测推理与生成之间的接口，而非只测两端。** 51.8% 那组说明有推理、无接口比没有推理更糟；做一次「去掉接口」的对照，能暴露系统是否真在用中间产物。
4. **给结构链留回退，至少留检测点。** 单向四轮的错误会累积，应记录每轮状态快照与评分，使失败可归因到具体某轮。
5. **自训练循环保留失败样本，并监控分布宽度。** 用多样性指标而非只看成功率，防止朝自身偏好收敛。
6. **严格区分代理指标与真实目标。** pLDDT、TM-score、EC3 都是计算层代理，以它们为奖励的优化必须配一条通往湿实验的验证通道。

## 关联

- 概念：[[concepts/ai-self-improvement-bootstrapping|自我改进引导]]、[[concepts/verifier-driven-development|验证驱动开发]]
- 相关实体：[[entities/claude-discovers-art-enzyme-system|Claude 发现 ART 酶系统]]、[[entities/boltzgen-protein-design-amazon-sagemaker-ai|BoltzGen 蛋白质设计]]、[[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026|AI4AI 综述与组合缺口]]、[[entities/jepa-anything-cross-domain-latent-world-model-phai-2026|PhAI JEPA-Anything 跨域潜世界模型]]

→ [[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026|原文存档]]
