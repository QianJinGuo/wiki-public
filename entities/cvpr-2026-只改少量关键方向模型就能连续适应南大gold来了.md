---

title: "CVPR 2026 | 只改少量关键方向，模型就能连续适应？南大GOLD来了"
created: 2026-07-05
updated: 2026-11-10
type: entity
tags: [ai, agent, llm]
sources: [raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CVPR 2026 | 只改少量关键方向，模型就能连续适应？南大GOLD来了

## CVPR 2026 | 只改少量关键方向，模型就能连续适应？南大GOLD来了

## CVPR 2026 | 只改少量关键方向，模型就能连续适应？南大GOLD来了

上线后的视觉模型，面对的从来不是一个静态测试集，而是一条持续变化的数据流。^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


  


自动驾驶摄像头会从白天驶入黑夜，从晴天进入雨雾；视频分析系统会不断遇到新的光照、视角和场景；医学影像模型也可能因为采集协议、设备型号或人群分布变化而遭遇域偏移。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


对于真实部署环境中的视觉模型来说，分布变化不是例外，而是常态。^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


  


这正是 Continual Test-Time Adaptation（CTTA）试图解决的问题：模型已经在源域完成训练，部署后无法再访问源数据，只能接收无标签的目标域数据流，并且每个样本通常只会按顺序经过一次。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


模型需要一边推理、一边适应新的分布，同时还不能因为适应过程中的噪声和漂移把自己“改坏”。^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


  


然而，在线适应天然存在一个矛盾：如果更新得太少，模型很难跟上持续变化的测试分布；如果更新得太多，推理成本会迅速上升，伪标签噪声会被不断放大，参数也可能在长程数据流中逐渐漂移，最终导致性能塌陷。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


换言之，CTTA 的核心难点并不只是“如何适应”，而是“应该在哪里适应、适应多少、以及如何避免越改越差”。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


本文从一个更直接的问题切入：如果完整特征空间既庞大又危险，那么是否存在一块更小、更关键、也更稳定的适应空间？ ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


我们发现，连续测试时适应并不一定需要在整个高维特征空间中反复调整；真正有效的更新方向，可能集中在一块由预训练分类器决定的低秩子空间中。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


我们将其称为 golden subspace，并进一步提出 Guided Online Low-rank Directional adaptation（GOLD），通过在线维护这一“黄金子空间”，让模型在部署阶段以更轻量的方式完成连续适应。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


大量分类与分割实验表明，GOLD 在效率、稳定性和整体性能之间取得了更好的平衡，为真实场景中的视觉模型部署提供了一种更高效、更稳健的测试时自适应方案。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


  


  


论文标题：

The Golden Subspace: Where Efficiency Meets Generalization in Continual Test-Time Adaptation ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

论文作者：

Guannan Lai, Da-Wei Zhou, Zhenguo Li, Han-Jia Ye^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


作者单位：

Nanjing University；Hong Kong University of Science and Technology；Frontier Robotics ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

录用会议：

CVPR 2026

论文地址：

https://arxiv.org/abs/2603.21928^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


代码地址：

https://github.com/AIGNLAI/GOLD ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


  
  


引言

视觉模型在实验室中通常面对的是固定测试集，但在真实部署中，它们遇到的是一条持续变化的数据流。^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]


  


自动驾驶系统会经历白天、黑夜、雨雾、逆光等复杂环境变化；安防与视频分析模型会遇到新的摄像头、视角、场景和人群分布；医学影像模型也可能因为扫描设备、采集协议或患者群体变化而产生域偏移。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


对于这些长期运行的视觉系统而言，模型能力不能只停留在训练完成的那一刻，而需要在部署过程中持续适应环境变化。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


这类问题与持续学习密切相关：模型需要在不断变化的数据分布中保持可用性，既要吸收新环境中的信息，又不能因为更新过程破坏已有能力。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


不同的是，在真实测试阶段，模型往往无法访问源域训练数据，也拿不到目标域标签，只能依赖连续到来的无标签样本进行在线调整。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


测试时适应（Test-Time Adaptation, TTA）正是围绕这一问题展开：模型在测试阶段利用无标签目标数据修正自身状态，从而缓解训练分布与测试分布之间的偏移。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


进一步地，Continual Test-Time Adaptation（CTTA）将 TTA 放到更加真实也更加困难的连续非平稳场景中：数据分布持续变化，样本按顺序到达，模型每一次适应都会影响后续状态。 ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

  


因此，CTTA 不是一次性的微调问

-> [[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了|原文存档]] ^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md]

---
## 深度分析

### 1. 低秩子空间：把 CTTA 矛盾重新表述为"在哪里更新"

CTTA 的老困境是在"更新太少跟不上漂移"与"更新太多放大伪标签噪声、推高成本"之间打摆。GOLD 的切入点是把问题从"更新多少参数"改写成"更新空间的几何结构是否正确"：对冻结特征提取器做单步最小范数分析可以证明，要以尽可能小的特征改动修正模型输出，最优更新方向不会散落在整个高维空间，而是落在由分类器权重行空间诱导的低秩子空间里^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:173-185]。AGOP 谱测量进一步支持这一点：前 64–128 个特征向量即可捕获超过 99% 的谱能量，说明低秩不是人为压缩，而是 CTTA 问题自带的结构^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:278-283]。这与 [[entities/mind-lab-lora-continual-learning-system|Mind Lab LoRA 体系]]用 0.12% 参数增量做持续学习共享同一直觉：可塑性不必等价于全参数更新。

### 2. AGOP 让子空间在无标签流上在线演化

只用源域分类器权重定子空间，得到的只是源域判别几何。直接在无标签流上反向训练分类器也不行：分类器是最末映射层，一次错误更新会沿"错误伪标签→错误更新→更错误伪标签"的链条累积放大^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:233-238]。GOLD 改用 Average Gradient Outer Product（AGOP）估计当前目标域中真正敏感的判别方向：AGOP 统计输入—输出梯度的外积平均，反映输出对各输入方向的敏感程度^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:243-253]；由于理想黄金子空间与分类器权重行空间相关，AGOP 既不需要源数据、也不需要目标标签，就能让子空间在线吸收目标域信息并快速逼近真实黄金子空间^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:263-278]。

### 3. Adapt/Update 双阶段：可塑性放进低秩适配器，稳定性交给冻结件

在线循环两步交替。Adapt 阶段：冻结 backbone 提特征并投影到黄金子空间，只学习一个紧凑 scaling vector 对低维坐标重标定，再以残差形式映射回原空间——接近单位向量时模型几乎保持原行为，需要时只在少量关键方向上动^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:303-313]。Update 阶段：筛选高置信样本，用 AGOP 更新子空间估计，同时以自训练损失加源域类别原型对比损失更新 scaling vector；源样本不可访问，但原型可在部署前预提取固定，作为语义锚点约束漂移^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:318-323]。设计本质是把可塑性放进低秩适配器，把稳定性留给冻结 backbone、固定分类器与源原型锚点^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:328]——与 [[concepts/catastrophic-forgetting|灾难性遗忘]]治理中的稳定性-可塑性权衡一脉相承，南大同组的 [[entities/nju-same-moe-continual-learning|SAME]] 也在 MoE 持续微调中处理同一矛盾。

### 4. 效率数字说明"省参数"与"性能"不冲突

分类基准上，GOLD 相对最强基线 OBAO 把 CIFAR10-C/CIFAR100-C/ImageNet-C 平均错误率从 14.6/29.5/59.6 降至 14.1/28.6/59.3^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:350]；每 batch 平均约 0.25 秒，接近最快的 SANTA 但性能更强——低秩适应不是单纯省参数，而是把在线优化预算集中到有效方向^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:357]。消融显示仅初始子空间已带来明显提升，AGOP 在线更新让子空间跟随目标域继续改进，原型对比损失补上长程语义稳定^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:364-369]；分割侧 CarlaTTA 的 highway 序列（外观与类别分布双偏移）达 34.5 mIoU，超过 CoTTA 的 33.8^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:383]。

## 实践启示

### 1. 先问"在哪里更新"，再问"更新多少"

把适应预算花在定位正确的低秩子空间上，往往比扩大可训练参数量更划算；作者给出的研究信号是：在线适应方法不必在参数量上内卷，而应关注更新空间的几何结构^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:410]。选型时优先评估"这个方法改的是哪块空间"，而不是只看可训练参数占比。

### 2. 无标签数据流上，别让噪声直接写进主干

冻结主干 + 低秩适配器 + 高置信样本筛选，把伪标签噪声的破坏半径限制在适配器内^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:328]。任何在线学习/自训练系统都值得复制这条防线：更新通道与语义主干隔离，错误更新可被后续正确样本覆盖而不污染全局。

### 3. 部署前固定语义锚点，部署后靠它兜底

源数据不可访问，但类原型可以部署前预提取并保存，在对比损失中充当锚点、防止特征空间漂移^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:323]。工程等价物：上线前把"语义参照系"物化保存，线上适应永远对着锚点校准，而非无约束自由漂移。

### 4. 持续学习不只发生在训练期，也发生在推理期

GOLD、[[entities/care-bi-level-routing-moe-continual-learning-hku-2026|CaRE]] 与 [[entities/nju-same-moe-continual-learning|SAME]] 分别在测试时适应、超长任务序列与 MoE 微调三个场景回答同一问题：如何在吸收新分布的同时不破坏已有能力^[raw/articles/cvpr-2026-只改少量关键方向模型就能连续适应南大gold来了.md:120]。对系统设计者的启示是把"持续适应"当作一等设计目标，为其单独规划参数预算、锚点与回滚机制。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

