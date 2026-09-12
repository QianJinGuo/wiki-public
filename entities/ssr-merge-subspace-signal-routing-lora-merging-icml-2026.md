---
title: "SSR-Merge：免训练 LoRA 合并的子空间信号路由（ICML 2026）"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [lora, model-merging, peft, diffusion, subspace, signal-routing, icml-2026, vivo, training-free, flux]
sources: [raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026]
confidence: 0.8
provenance_state: extracted
---

# SSR-Merge：免训练 LoRA 合并的子空间信号路由（ICML 2026）

SSR-Merge（Subspace Signal Routing，子空间信号路由）是 vivo BlueImage Lab 与南京大学等团队提出、被 ICML 2026 接收的免训练 LoRA 合并框架。它要回答的问题是：多个 LoRA 合并为什么会互相干扰（画风串味、角色崩坏），能不能不靠反复试参数，而是把干扰直接从数学上解出来。现有方法（Task Arithmetic 的参数直接相加、TIES/DARE 等稀疏化剪枝）本质上都在参数空间里做算术，冲突在「你中有我、我中有你」的纠缠状态下被取舍；SSR 换了一个根本思路——不在参数上做算术，在信号上做路由。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

## 方法：合并通道 → 解混 → 导航

SSR 分三步构造路由器 R = QG⁻¹。第一步「合并通道」：把 K 个 LoRA 的下投影矩阵沿秩方向拼接、上投影矩阵横向拼接，构造统一子空间，让所有任务信号走同一条加宽通道——这既是串扰的物理基础，也是路由的前提。第二步「解混」：用一小批校准数据跑一次前向，统计通道内各路信号的相关结构得到相关矩阵 G，乘上 G⁻¹ 即按统计规律把缠绕的混音反向拆开、还原彼此独立的成分。第三步「导航」：用由校准数据二阶统计量算出的方向引导矩阵 Q，把解混后的第 i 路信号对准第 i 个 LoRA 的出口，不让它漏进别人的通道。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

## 为什么可信：解析解而非启发式

SSR 最关键的差异是路由器「不是设计出来的，是推导出来的」：论文证明 R = QG⁻¹ 在数学上恰好等价于普通最小二乘（OLS）估计量的投影，即「让合并后各任务特征重建误差最小」的唯一解析解；传统任务算术（Task Arithmetic）则是该框架中 R = I（不做任何路由）的特例——老方法不是错了，而是放弃了信号调节这一自由度。工程上有三点让方法真正可用：单样本校准（每个任务只需一条代表性提示词、前向传播一个时间步，无需训练数据集或真值图像）、流式计算（利用充分统计量可加性边读边累加协方差，内存复杂度降到常数级 O((Kr)²)）、结构化重参数化（部署时把 R 直接吸收进上投影矩阵，合并产物结构上就是一个标准 LoRA，推理零额外开销、与现有生态完全兼容）。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

## 实验与效率

在 FLUX.1-dev 上合并 9 个 LoRA（覆盖全部 transformer 层），合并阶段耗时 34.26 秒，比稀疏化方法 TIES 快约 2.6 倍，与纯算术方法 DARE（20.95 秒）同一量级——即「以算术方法的速度获得显著更好的保真度」。三组层层递进的评测给出量化结论：单任务保真（K=9 最严苛设置）SSR 的 DINOv2 相似度 0.6713、CLIP 分数 0.7850，分别高于最强基线 IterIS 的 0.6240/0.7520，且从 K=3 到 K=9 始终保持单任务上限 90% 以上的恢复率（90.2%–98.6%），而基线随 K 增大明显滑坡；多任务同图组合（一句提示词同时出现 2–4 个主体）成功率 91%，比 DARE 高 29 个百分点（TIES/DARE 仅 69%/62%，靠「丢掉一部分任务」换表面稳定）；人像精修基准（口红、腮红、眼影三个独立编辑 LoRA 合并一次上妆）中 SSR 以均衡强度与准确位置最接近商业修图软件串行执行的真值。附录进一步给出 K=21 仍保持 77% 以上恢复率、从哩布哩布（Liblib）平台下载真实社区 LoRA（打光/人像美化/人像风格化）合并时 CLIP 0.821 为全部方法最高，以及在扩散模型之外的 GLUE 语言理解基准上以 80.9 平均分超过所有合并基线。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

## 局限与团队判断

SSR 优化的是局部线性重建目标，理论上不保证完整非线性扩散过程的全局最优（实验里该局部最优已非常接近上限，但极端条件下差距仍可能拉大）；当被合并任务之间存在严重域冲突或高度语义重叠时，信号本身难以区分，路由难度上升、性能可能下降；更高保真的多概念组合能力同时意味着更强的合成内容生产力，存在被误用风险。团队对这项工作的两点判断值得记录：其一，LoRA 合并是「能力资产化」的关键一环——社区数以万计的 LoRA 是真正的资产，但资产只有能组合才有复利，SSR 证明合并可以是无训练、有理论保证、零额外推理开销的标准操作（下载即可合并、合并即可上线）；其二，模型合并领域该从「调参的艺术」走向「统计估计的科学」。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

## 关联

在 LoRA 生态中，SSR-Merge 补齐了「多个 LoRA 如何无损组合」这一环，与 [[entities/geora-geometry-aware-lora-rlvr-meituan-2026|GeoRA（面向 RLVR 的几何感知 LoRA）]] 关注的「单 LoRA 如何训练得更好」形成互补：前者是推理/部署期的合并问题，后者是训练期的适配问题。其「把干扰重新表述为信号空间的估计问题」的思路，与 [[entities/macaron-v1-lora-moe-architecture|Macaron v1 的 LoRA-MoE 架构]] 在结构上做专家路由有相通之处——都是把「共享参数空间里的互相干扰」转化为「显式的路由/选择」。相关模型资产与蒸馏/适配的工程背景可对照 [[entities/glm-5-2-mixed-lora-200m-context|GLM-5.2 Mixed-LoRA 长上下文]]。^[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026.md]

→ [[raw/articles/ssr-merge-subspace-signal-routing-lora-merging-icml-2026|原文存档]]
