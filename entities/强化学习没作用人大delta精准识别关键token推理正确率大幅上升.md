---

title: "强化学习没作用？人大DelTA精准识别关键token，推理正确率大幅上升"
type: entity
created: 2026-07-02
updated: 2026-09-19
tags: [wechat, ai]
rating: v9c9
sources:
  - raw/articles/强化学习没作用人大delta精准识别关键token推理正确率大幅上升
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 强化学习没作用？人大DelTA精准识别关键token，推理正确率大幅上升

## 摘要

中国人民大学高瓴人工智能学院团队提出 token credit assignment 算法 **DelTA**（**D**iscriminative signa**l**-guided **T**oken credit **A**ssignment）。它不再依靠经验或直觉手工分配 token 权重，而是通过求解优化问题为 RL 目标中的每个 token 计算最优权重。论文从一阶泰勒近似出发，证明主流 RL 目标对 token 概率的更新本质上等价于一个**线性判别器**，DelTA 正是通过放大正负质心的可分性来改进训练。 ^[raw/articles/强化学习没作用人大delta精准识别关键token推理正确率大幅上升.md]

## 核心要点

- **问题定位**：大模型 RL 微调长期存在训练不稳定、正负样本梯度难以区分的问题；行业内普遍依赖手工经验为 token 分配权重，始终拿不到最优训练效果。
- **核心方法**：DelTA 不依赖经验或直觉，而是**求解优化问题**来得到 RL 目标中每个 token 的最优权重，迭代式地交替更新「token 权重」与「正负质心」。
- **线性判别器**：RL 会隐式地把 token 分成两堆，正/负 advantage 各对应一个质心；token 梯度和哪个质心更接近，就提高还是降低该 token 的生成概率。看似复杂的 RL 目标实际在做线性分类工作。
- **加权动机**：标准 DAPO 对所有 token 一视同仁，但正误答案文本大量重叠，重叠 token 会稀释正负质心的区分度；给有区分度的 token 更高权重，正负质心就能被推得更远。
- **实验覆盖**：适用于几乎所有主流强化方法与主流强化框架，在数学推理、代码生成、知识问答等 10 余个任务上，对不同尺寸、不同类别的 base 模型带来显著提升。
- **量化增益**：在 7 个数学推理任务上相较最强基线分别提升 **3.26**（8B）与 **2.62**（14B）；对比对象包括 DAPO、DAPO with forking tokens、SAPO 与较新的 FIPO。
- **反直觉现象**：已有算法在提升 reward 的同时往往让 token 熵变大（更鼓励探索），DelTA 在同样拿到可观 reward 提升的同时 **token 熵反而下降**，暗示训练可能更稳定。
- **权重验证实验**：按 DelTA 权重排序后只训练前 50% 高权重 token，效果超过随机 50% 甚至超过全量 DAPO；只训练后 50% 低权重 token 则很快崩溃——说明它在筛选真正有学习价值的梯度，而非简单稀疏化。

## 深度分析

### 手工 token 权重的局部最优陷阱

传统做法把 token 权重当成超参数或启发式规则来设计。其根本缺陷在于：设计者的目标是「让训练好看」，而不是「让正负样本在梯度空间里更可分」。权重由人拍板时必然落在先验决定的局部邻域内，无法随 rollout 分布、模型尺寸、任务类型自适应变化。文章开篇的两个症状——训练不稳定、正负样本梯度难区分——其实是同一病因的两种表现：正负质心被大量共享 token 拉近，判别边界模糊，梯度信号既弱又互相干扰。正确的提问方式因此不是「哪些 token 该加权」，而是「什么样的权重能让判别器最可分」——一个可形式化求解的优化问题。 ^[raw/articles/强化学习没作用人大delta精准识别关键token推理正确率大幅上升.md]

### 一阶泰勒分析：RL 更新其实是个线性判别器

研究团队的分析起点是对待生成 token 的概率（记 x 为待生成 token、c 为已生成上下文）做一阶泰勒近似，把概率变化拆成「token 梯度 · 参数变化」两项。把 DAPO 的优化目标代入参数变化的表达后整理，可以定义出正、负两组 advantage 对应的质心。结论很干净：RL 隐式地把 token 分成两堆，更新机制就是拿 token 梯度和两个质心做比较，靠近正质心则提高生成概率，靠近负质心则降低生成概率。这一视角把 RL 从「策略优化黑箱」降维成「线性分类器」：分类性能取决于两类质心的间距，而间距又由 token 的加权方式决定——这直接指出了改进的着力点。文章强调讨论虽以 DAPO 为对象，结论都可推广到形式相近的主流 policy optimization 方法。

### 从「拍脑袋」到最优权重：DelTA 的三步迭代

DelTA 把上述洞察变成算法：第一步，固定当前正负质心，求解优化问题得到每个 token 的最优权重——直观上，若某 token 对应正 advantage（来自正确答案），优化会希望它离正质心更近、离负质心更远，负 advantage 侧对称定义；第二步，用求得的权重对 token 加权重新计算质心，权重越大代表区分度越大，对质心位置的影响也越大，由此正负质心被推得更远；第三步，迭代收敛后把最终权重代入 RL 目标，正常运行强化学习算法。这不是外加的正则项或 curriculum，而是对已客观存在于 RL 更新内部的判别器做显式优化——权重与质心互相定义、交替求解，类似 EM 式的坐标上升。文章坦承：为效率起见，当前在 token 梯度上做了非常大幅的近似，这会限制 DelTA 的性能上限。 ^[raw/articles/强化学习没作用人大delta精准识别关键token推理正确率大幅上升.md]

### 跨算法、跨模型、跨任务的泛化与分析的边界

实验矩阵刻意做成「三向交叉」，以证明收益不是特定配置的偶然：算法侧对比 DAPO、DAPO with forking tokens、SAPO、FIPO；模型侧从 Qwen3-8B-base、Qwen3-14B-base 扩展到 Allen Institute 的 Olmo3-7B-base，验证不依赖基模选择；任务侧覆盖 7 个数学推理基准（AIME24/25/26、HMMT25 两场与 HMMT26 等）、代码生成（HumanEval+、MBPP+、LiveCodeBench）与知识推理（GPQA-Diamond、MMLU-Pro）。其中把数学数据上训练的 Qwen3-8B-base 直接迁移到 GPQA-Diamond 与 MMLU-Pro 仍获提升，说明更好的 token 权重带来的是可迁移的能力增益而非过拟合。论文也划出了边界：团队已从数学上证明 DelTA 不依赖具体强化方法与 verifiable reward，因此向更大模型、更多在线算法推广是自然的下一步；更高效、理论上更合理的梯度近似则是工程上的主要待解问题。整套工作由人民大学高瓴人工智能学院二年级硕士张凯翼担任第一作者，论文编号 arXiv 2605.21467，代码开源于 RUCBM/DelTA。

## 实践启示

1. **把 token 权重当成可优化变量而非超参数**：检查你的 token 加权是否来自启发式规则，把它替换为「使正负质心最大化可分」的优化目标。
2. **用「只训高权重 token」当免费的诊断器**：按权重排序，分别用前 50%、随机 50%、后 50% 计算损失做对照。若前 50% 能超过全量、后 50% 立刻崩溃，说明权重真的在筛选学习信号；若三者差别不大，说明你的 credit assignment 没有起作用。
3. **同时监控 reward 与 token 熵**：熵随 reward 一起上升通常意味着模型在靠扩大探索换分数；DelTA 报告的是「reward 升、熵降」的组合，可作为训练是否更稳定的辅助信号，也呼应了 [[entities/rlvr-entropy-collapse-steer-acl-2026-outstanding|token 熵坍塌]] 这条工作线。
4. **在迭代步数上做精度/成本折中**：DelTA 的关键超参是权重–质心交替更新的步数与梯度近似精度。近似越粗越省但上限越低；先在小模型上标定「迭代步数—收益」曲线，再迁移到大模型。
5. **模型与算法都要换着试**：既然结论不依赖具体 policy optimization 方法，接入时优先做「同基模换算法」与「同算法换基模」两组交叉验证，避免把收益误归因于特定配置。
6. **跟进开源实现的后续版本**：当前最大短板是 token 梯度的强近似，跟进 RUCBM/DelTA 的后续梯度估计改进比自行重实现更划算。

## 相关链接

- RLHF/DPO/GRPO 对齐
- [[concepts/reinforcement-fine-tuning-rft|强化微调 RFT]]
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR：可验证奖励强化学习]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进：PPO→DPO→GRPO]]
- [[entities/agent训练最容易踩的坑credit-assignment-is-all-you-need|Credit Assignment 是 Agent 训练最大的坑]]

→ [[raw/articles/强化学习没作用人大delta精准识别关键token推理正确率大幅上升|原文存档]]
