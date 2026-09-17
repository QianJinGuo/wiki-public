---
title: "QuantaMind：分子之心的反应性机器学习力场（Science Advances）"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [ai4s, mlff, molecular-dynamics, force-field, enzyme-catalysis, science-advances, moleculemind, transition-state, active-learning]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026]
---

# QuantaMind：分子之心的反应性机器学习力场（Science Advances）

分子之心（MoleculeMind，许锦波团队）自研的**反应性机器学习力场 QuantaMind**，以接近 DFT 的精度、与经典分子动力学相当的速度，在万原子级体系上稳定模拟了完整酶催化循环，并在内部产业项目中把尺度推到十万原子、百纳秒；论文登上《Science Advances》。十万原子级反应体系的单步模拟耗时，从传统方法的千万天量级压缩到 **0.25 秒**。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 为什么「反应性模拟」是地狱难度

常规分子动力学（MD）能模拟分子结构如何运动——蛋白质怎么折叠、配体怎么结合、构象怎么变化——但原子之间的成键关系是预先固定的，不会自行断裂或形成。这意味着常规 MD 能告诉你「分子怎么动」，却无法告诉你「反应怎么发生」；而酶催化、质子转移、配体结合后的构象变化、药物分子与靶点的共价作用，恰恰是研发中最关键的机制问题。要模拟这些过程需要**反应性分子动力学**，难度比常规 MD 高出一个数量级。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

三个维度同时拉满：^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

- **精度**：化学反应涉及电子结构变化，只有量子力学方法（如 DFT）才能准确描述成键与断键，但 DFT 成本是 O(N³) 甚至更高，体系稍大就完全算不动；
- **时间尺度**：很多化学反应是罕见事件——水的自解离可能要跑几纳秒才出现一次，酶催化限速步骤需要更久；模型必须在数十万至数百万时间步的连续推演中保持稳定，不能跑着跑着就崩；
- **空间尺度**：真实生物体系包含蛋白质、底物、离子和大量显式水分子，动辄数万乃至十万原子。

机器学习力场（MLFF）用神经网络拟合量子化学数据、以推理代替每步电子结构计算，理论上可兼顾精度与速度，但存在一个致命问题：**单个构型上预测准确，不等于能完成可靠的长时间模拟**。很多 MLFF 短时表现不错，一跑长轨迹就出现能量漂移、构型崩坏，最终只能算小分子、短时间。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 三个关键设计

QuantaMind 的关键不在于堆更大的模型，而在于围绕「反应性」重新设计数据组织与训练方法：^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

- **以过渡态为中心的训练数据**：许多 MLFF 主要从平衡或近平衡构象学习，模型没见过「过渡态长什么样」，遇到断键成键就露怯。QuantaMind 重点纳入非平衡构型和以过渡态为中心的反应数据，论文用 **5286 个酶催化过渡态构型**训练模型，让模型「真正见过」键正在断裂、正在形成的瞬间。
- **DFT 方法分类嵌入**：不同泛函和基组的量子化学数据精度不同，传统做法要么只用一种精度（浪费其他数据），要么简单混合（模型分不清数据质量）。QuantaMind 把计算设置编码为模型可识别的类别特征统一纳入训练，让模型能区分并利用不同精度层次的数据。
- **主动学习式迭代闭环**：面对新体系先跑轨迹，识别模型尚未充分覆盖的构型，补充 DFT 计算继续训练；每一次新任务不仅解决具体问题，也沉淀可复用的反应数据，持续扩大可可靠处理的体系与反应类型，形成自我强化的数据飞轮。

稳定性有硬指标：1ns NVE（微正则系综）模拟中，纯水体系的总能量漂移只有 **10⁻⁵ kcal/mol/atom/ps** 量级，可稳定支撑数十纳秒的长时间模拟。数值虽小，但在 MLFF 领域意味着模型真正通过了「长时间动态稳定性」的考验。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 四个验证数字

**偏差降低 87%。**质子扩散系数是对长时间反应性模拟能力的严苛检验——它不是一次跳跃的速度，而是质子沿氢键网络在水分子之间上万次「接力」后形成的统计结果。QuantaMind 对六种尺寸水体系各做 1 纳秒模拟，最大体系 24001 原子，每条轨迹观察到上万次质子转移，预测扩散系数 **0.87±0.10 Å²/ps**，与实验值 0.94、0.96 高度吻合，相比此前同类 MLFF 研究（Huo et al., 2023）偏差降低约 87%。更关键的是模型并未专门训练过水合氢离子，它凭「通用反应能力」自己学会了质子跳跃。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

**AI 自己算出了水的 pH。**水的自电离是极罕见事件，很难在有限模拟中直接捕捉。QuantaMind 在 373K 下对 9,999 原子体系模拟 3 纳秒，捕获多次自发电离事件，预测平均 pH **6.34±0.06**，与同温度实验值 6.13±0.03 接近。模型并未预先写入传统恒 pH 模拟中的经验酸碱规则，而是从原子级反应动力学中自然涌现出与实验接近的宏观 pH；又通过 100 条独立模拟估算酸碱中和速率，与实验值处于同一数量级。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

**pKa 5.81，与 NMR 实验值 5.5 接近。**把溶菌酶放入真实磷酸盐缓冲液模拟 20 纳秒，让组氨酸自己「决定」质子化，缓冲液中磷酸盐物种的电离、水解与酸碱平衡也被一并真实模拟；估算 His15 pKa 约 **5.81**，与核磁共振实验值 5.5 接近。这意味着研究者无需事先假定残基状态，模型便能模拟其随酸碱环境的动态转换。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

**17792 原子、完整催化循环、原子力分量 r>0.99。**对由 257 残基 PET 水解酶、1 个 MHET 底物和 4898 个水分子构成的 17792 原子体系做全原子反应性模拟，所有原子（包括溶剂水）统一显式处理，无需 QM/MM 式区域切分。在 6 纳秒轨迹中完整模拟了酰化、脱酰化和催化位点恢复全过程。限速步骤能垒 **14.5±0.1 kcal/mol**，落在实验 LCC 类酶的 14.5–16.5 区间；从轨迹中抽取 **342 个构型**做独立 DFT 单点验证，原子力分量 Pearson 相关系数均超过 **0.99**。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 从论文到产业

论文验证的是 1.8 万原子、6 纳秒，但内部测试的推理速度已达约 **2.1×10⁻⁶ 秒/原子/步**（单张 A100 80GB），较论文版本提升近一倍；模拟体系扩大到十万原子级、时长拓展到百纳秒级。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

两个已跑通的产业方向：**酶工程**把传统「改了再测」变成「先算再测」——先在计算端比较不同突变如何改变质子转移路径、反应能垒和关键步骤，更有针对性地优选突变位点、缩小实验范围；**抗体设计**中的 pH 敏感性抗体项目中，QuantaMind 与自研 AI 模型协同，模拟目标 pH 区间内关键残基的状态切换，得到的候选抗体在 pH 6.0 下的解离速率达到 pH 7.4 下的 **62 倍**。相关功能已集成到分子之心的 AI 原生生物「智」造操作系统 MoleculeOS（MOS），构成「生成设计—结果预测—机制模拟」闭环中的**机制评估引擎**。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 团队背景

许锦波 2016 年提出的 RaptorX-Contact 第一次证明 AI 能有效预测蛋白质结构，比 AlphaFold1 早两年；2022 年他回国创办分子之心。结构预测回答的是「蛋白质长什么样」，而蛋白质如何催化反应、如何与配体动态结合、如何在不同 pH 下改变状态，这些「怎么工作」的动态过程才是决定药物能否起效、酶能否高效催化的关键，也是计算化学几十年来未补齐的缺口。^[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026.md]

## 要点

- **「以过渡态为中心的数据组织」是反应性 MLFF 与普通 MLFF 的分水岭**——只从平衡构象学习的模型在断键成键处必然失手；
- **长时间稳定性（能量漂移量级）比单帧精度更能决定可用性**：10⁻⁵ kcal/mol/atom/ps 是「能连续生成一部物理合法的原子电影」的门槛；
- **DFT 方法分类嵌入**把异质精度的量子化学数据从负担变成可加权的资产，是数据侧而非模型侧的创新；
- 宏观量（pH、pKa）从原子级动力学中自然涌现，而不是靠经验规则注入，这是「机制模拟」区别于「参数拟合」的证据。

## 相关

- [[entities/moleculeos-ai-biology-operating-system-xujinbo-2026]]
- [[entities/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026]]
- [[entities/deeppotential-alibabacloud-agentrun-scientific-ai]]
- [[entities/ai4s-2026-h1-frontier-panorama-yinxi]]
- [[entities/scienceclaw-autoproject-project-level-ai4s-zidong-taichu-2026]]
- [[entities/from-code-to-molecules-an-ai-driven-egfr-inhibitor-discovery-journey]]
- [[entities/boltzgen-protein-design-amazon-sagemaker-ai]]
- [[entities/mira-mpa-deep-principle-ai4s-40-sota]]

→ [[raw/articles/quantamind-reactive-mlff-moleculemind-science-advances-2026|原文存档]]
