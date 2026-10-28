---
title: "TRACE：Rollout 引导的 FP4 量化感知训练，让 MoE 强化学习又快又稳"
created: 2026-10-24
updated: 2026-10-26
type: entity
tags: [rl, quantization, fp4, moe, post-training, inference-optimization, qwen, arxiv]
sources: [raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767]
confidence: 0.78
provenance_state: extracted
---

# TRACE：Rollout 引导的 FP4 量化感知训练，让 MoE 强化学习又快又稳

阿里千问（Qwen）与俄亥俄州立大学联合提出 TRACE（arXiv:2610.07767），要解决的是强化学习里最贵的一段——Rollout——能不能压到 FP4 还不变崩。结论是能，但前提不是"把两边都量化得更准"，而是**把训练侧与 Rollout 侧的低精度路径差异直接压小**。在 Qwen3.5-35B-A3B 上平均分 75.3（BF16 Rollout 74.9），8K 输入 / 128K 输出时解码吞吐相对 BF16 Rollout 最高 5.4×，端到端 RL step time 只多约 7.4%。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 问题：FP4 Rollout 为什么把 RL 训练"搞崩"

Rollout 阶段在长轨迹 RL 里占掉大量时间，把 BF16 换成 FP4 能同时省算力和显存，但现有做法（普通 FP4 QAT、QaRL、QUADS）在训练后期普遍出现奖励与测试分数下滑甚至崩溃。TRACE 作者给出的根因不是"量化误差大"，而是**误差度量选错了对象**： ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

- 已有方法都在降低"量化后的值离原始高精度值有多近"，即分别优化训练路径和 Rollout 路径各自的量化误差；
- 但两条路径各自都量得很准，仍可能被量化到**不同码点**,于是微小差异被放大。论文的例子：训练侧激活 60.24、Rollout 侧 59.76，原始只差 0.48；经同样的 global scale + block scale 归一化后为 2.51 与 2.49，恰好落在舍入边界两侧，一个量化到 3、一个量化到 2，最终差异放大到 24；
- 对 MoE 更致命：数值扰动会传导到 router 分数与专家选择，训练策略与实际生成轨迹对应的策略进一步偏离——这正是 [[concepts/moe-mixture-of-experts-2025|MoE]] 结构下低精度 RL 的放大器。

## 机制一：让训练侧向 Rollout 的量化结果对齐

Rollout-Guided QAT 的核心动作：Rollout 阶段记录 **FP4 routed expert 激活**与 **FP4 KV 状态**对应的量化结果；训练阶段这些信息不只当历史数据，而是直接参与训练侧 fake quantization 的**舍入决策**。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

- 训练侧仍只在相邻两个 FP4 E2M1 码点之间选，但不再机械用 round-to-nearest（RTN）；
- 改为优先选择**距离 Rollout 侧实际码点更近**的那个候选值。

这把"两条路径互相靠近"变成了一次显式的舍入方向对齐，而不是两个独立误差最小化问题。

## 机制二：只传"最少必要"的量化信息

完整保存 Rollout 量化结果要经历 GPU→CPU→缓存→磁盘→回 GPU，长轨迹、大 batch、异步 RL 都会放大存储与通信。作者的统计给了捷径：**超过 99% 的训练/Rollout 量化不一致只相差一个相邻 FP4 codebook entry**。既然训练侧自身激活已把候选范围压到极小，Rollout 侧只需保留**少量 mantissa 信息 + scale**，就足以告诉训练侧往哪个方向舍入。实测中若保存完整指导信息，Rollout 会被通信开销严重拖慢；改用紧凑缓存后吞吐贴近原生 FP4 Rollout。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 机制三（意外发现）：模型会在 RL 中"适应"FP4

训练早期 FP4 执行仍有性能损失，但只要 train-rollout discrepancy 被控制住，模型会逐步适应这个低精度执行环境： ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

- Qwen3.5-35B-A3B 在 HMMT25 曲线上约 **120 个 RL step** 后追平 BF16；
- Qwen3.8-Flash-Next 上类似"追平"出现在约 **240 step**。

含义是：低精度不必只被当作静态噪声。若训练与生成处在**一致、可学习的低精度环境**中，策略本身会通过 RL 更新适应这种数值约束。这也解释了为什么"先 BF16 训练再事后量化"打不过"训练期就用 FP4"——后者最终 FP4 policy 平均分 75.3，而 Vanilla NVFP4 / 4over6 / H-Scale 事后量化分别是 70.4 / 71.0 / 71.4。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 数据与边界

| 设置 | 结果 |
|---|---|
| Qwen3.5-35B-A3B 平均分 | TRACE 75.3 / BF16 Rollout 74.9 / QUADS 68.8 / QAT 59.6 / QaRL 58.6 |
| HMMT25 | 从 QUADS 的 59.0 提到 70.0 |
| 解码吞吐（8K in / 128K out） | 相对 BF16 Rollout 最高 5.4× |
| 端到端 RL step 开销 | 约 +7.4% |
| 覆盖模型 | Qwen3.5-35B-A3B、Qwen3.5-122B-A10B、Qwen3.8-Flash-Next、Qwen3.8-2.4T-A95B |

边界要看清：结论建立在 MoE 架构 + FP4 E2M1 码点 + 自家 RL 任务栈上，128K 输出下的吞吐优势并不等于所有轨迹长度都同样受益；"模型适应低精度"目前是曲线观察，缺少对失败模式（何时不追平）的系统刻画。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 可迁移的三条

1. **一致的低精度环境优于事后量化**：训练与推理用同一套低精度语义，模型会把量化边界当约束学进去，而不是当噪声扛过去。
2. **差异度量和误差度量不是一回事**：两个系统各自误差小，仍可能互相错位；对齐"两条路径的相对位置"往往比叠加两个绝对误差更有效。
3. **传输成本用分布统计砍**：99% 的不一致只跨一个码点，这个统计事实直接决定了紧凑缓存方案可行——先量化"信息需要多精确"，再设计协议。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 深度分析

### 一、把量化从「部署压缩」改成「训练环境的一部分」

通行流水线是先用 BF16 做 RL 后训练得到一个名义策略，再用 PTQ/QAT 把它近似成交付版本，量化因此被视为一个可以用后验指标（per-tensor 误差、重构损失、少量校准集上的困惑度）独立评价的压缩步骤。TRACE 拒绝这个顺序的理由是 RL 特有的：RL 的目标不是单步似然而是沿轨迹累积的奖励，策略的质量由它自己采样出来的轨迹定义；一旦"生成轨迹的模型"和"计算优势的模型"不是同一个函数，[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|重要性比率]]就不再只度量策略漂移，而是度量策略漂移与架构漂移的混合物。所以真问题不是"FP4 近似得够不够好"，而是"近似后的模型还是不是被优化的那个模型"。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

这个顺序反转买到的是：量化误差从一个不可消除的固定偏置，变成可被策略更新吸收的约束。事后量化里，量化误差是训练目标从未见过、也无法通过梯度抵消的外生扰动；TRACE 把 FP4 码点写进 rollout 的转移核，策略在 RL 中收到的奖励梯度天然携带"在这个码点网格上该如何行动"的信息。这恰好解释了 120 / 240 step 的追平曲线，以及 75.3 对 70.4 / 71.0 / 71.4 的事后量化差距——后者并非量化算法更弱，而是它们的策略从未在低精度环境里被优化过。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

代价是部署自由度与栈耦合。其一，训练时的码本、block scale 粒度、KV 精度成为策略定义的一部分：换 kernel、改 block size、调 TP/EP 度都会移动 scale 分组，等价于换了一个策略，已获得的适应随之失效，而这种失效表现为行为漂移而非单调掉点，比"量化掉分"更难诊断。其二，rollout 引擎从"可替换的采样器"变成训练语义的一部分，训练侧与 rollout 侧的 fake-quant 必须共享舍入语义，任何一侧 kernel 的差异（融合算子中 scale 的计算顺序、累加精度）都会直接变成训练方差。其三，失去"一个 checkpoint 服务多个精度档位"的能力——想同时交付 FP8 与 FP4 版本，需要两套训练或至少两套对齐。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

### 二、FP4 的扰动为什么专打 MoE 的 router

FP4 E2M1 的有效幅度只有 0、±0.5、±1、±1.5、±2、±3、±4、±6 这几档，在 |x| ∈ [0.5, 2] 区间相对分辨率最差，而这正是 post-norm 激活最密集的地方。60.24 与 59.76 只差 0.48、相对误差 0.8%，看似无害；除以相同的 block scale 后落在 2.51 / 2.49，恰好跨在 2.5 的舍入边界两侧，一个进 3、一个进 2，相对误差变成 20%，乘回 scale 就是 24 的绝对差。这里的关键不是"FP4 精度低"，而是映射是阶跃的：输出差异是输入差异的非连续、非线性函数，缩小输入差并不保证缩小输出差，误差度量与差异度量因此不存在单调对应关系。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

router 是全网最脆的一层，原因不在它算得准不准，而在它下游接的是 top-k 选择而非加权求和。专家输出对 router logit 是连续的，top-k 却是离散的：logit 上 δ 的扰动不会产生 δ 量级的输出误差，而是以一定概率整体替换参与计算的专家集合，输出直接跳变成另一个函数的值。因此 [[concepts/moe-mixture-of-experts-2025|MoE]] 的脆弱性可以概括为"分数的不准被离散化放大成函数替换"。这也解释了普通 QAT 的 per-tensor 重构误差看起来正常、RL 后期却崩溃：指标度量的是激活重构，崩溃发生在专家选择，两者之间没有可微的桥梁。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

MoE 还有两级放大器。第一级是负载均衡与 capacity factor：专家选择翻转会改变每个专家收到的 token 数，触发 token dropping，而被丢的 token 既不计前向也不得梯度，于是下一层看到的 token 分布同样改变——一次舍入翻转可以跨层、跨 token 传播。第二级是 FP4 KV 的持续性：KV 一旦量化就被整条轨迹反复读取，误差不是每步独立重采样，而是沿 128K 输出单调累积，且已写入的 KV 无法回滚；这比 per-token 独立噪声严重得多，也解释了长输出这个 5.4× 收益最大的场景为何同时是误差最容易积累的场景。TRACE 用 rollout 侧真实码点做指导，本质是让训练侧 router 看到与 rollout 相同的专家结构，把架构漂移从重要性比率里摘出去，剩下的才是 PPO clipping 能正常处理的策略漂移。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

### 三、Rollout 兼作校准集：真正的价值是 on-policy

传统量化校准集是静态的几百条代表性样本，与策略当前访问的分布最多"大致相似"。TRACE 的信息通道不同：每条轨迹都在策略当前分布上产生 routed expert 激活与 KV 的成对张量，并且记录的是这些张量在真实 FP4 执行下的码点。这不是"更多校准数据"，而是"on-policy 校准数据"——校准分布自动跟随策略漂移，不需要人工设计域覆盖。这也解释了它为何能压住训练后期的崩溃：静态校准集在策略漂移后必然失配，而失配恰恰是后期才暴露的，早期两者几乎无差别，这也是此类方法在短程实验里看不出价值的原因。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

但校准目标是"一致"，不是"正确"。指导信号来自 rollout 侧的量化值，而 rollout 侧自身可能是偏的（典型如 KV 沿长轨迹累积的量化漂移）。TRACE 把训练侧拉向 rollout 侧，消掉的是两条路径之间的方差，而非 rollout 路径内部的偏差；若 rollout 路径存在系统性偏移，TRACE 会忠实地把这个偏移固化成训练目标。这与"平均分 75.3、BF16 74.9"的读数是自洽的：方法的目标是追平而非超越，超过 BF16 的部分本就不在它的优化范围内。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

更实际的风险是 off-policy 陈旧与分布错位。异步 RL 中产生指导的 rollout 可能落后数个策略版本，指导会把训练拉向旧策略的舍入模式；紧凑缓存设计自带部分缓解——99% 的不一致只跨一个相邻码点，因此一份"旧"的指导通常仍在正确方向上，但这是分布假设而非保证：当策略剧烈变化（训练早期、大学习率、高奖励方差）时旧码点可能已不在当前候选对之内，只能退回 RTN，静默退化为普通 QAT，论文未给出该回退率随 staleness 的变化。另一重错位来自 rollout 与推理：rollout 高温、长 CoT、探索性强，线上推理更集中、更短；且线上若改变 batch 形状、TP/EP 度或 kernel，block scale 的分组随之移动、码点整体平移，训练期学到的对齐方向只对训练时的 scale 布局有效。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

### 四、这些数字能证明什么，不能证明什么

能证明的是定性结论：同一任务套件上 TRACE 75.3 对齐 BF16 74.9，而 QUADS 68.8、普通 QAT 59.6、QaRL 58.6，6.5 到 16.7 分的落差远超随机种子波动，"一致性度量优于误差度量"这一点被稳固支持；HMMT25 从 59.0 到 70.0 同向印证，且四个 MoE 模型（35B-A3B / 122B-A10B / Qwen3.8-Flash-Next / 2.4T-A95B）都跑通，说明不是单点偶然。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

不能证明的比能证明的更多。HMMT25 是单一竞赛数学基准、方差大且未报种子与置信区间，11 分不应读作能力提升；5.4× 是 8K 输入 / 128K 输出下的解码吞吐最好情况，prefill 在这个配比下占比很低，真正决定训练成本的是端到端 step time，而 +7.4% 的口径是"带指导的 FP4 rollout 对不带指导的 FP4 rollout"，并非 BF16→FP4 的端到端加速比，这个数字全文未单独给出。缺失的对照同样关键：没有 [[concepts/inference-optimization|推理优化]] 实践中最常用的 FP8 rollout baseline（它以远低得多的风险拿到大部分显存收益），没有与"缩短 rollout 长度 / 改进采样 / 蒸馏小模型"这类更便宜的路径比较，也没有给出等 wall-clock 下的精度-算力前沿——即 TRACE 的 FP4 是否打得过同等时间的 BF16，而这才是选型的实际问题。"模型适应 FP4"目前只是两条曲线，缺少不追平情形（小模型、dense 架构、极短 RL、高学习率）的系统刻画。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

对没有同等算力预算的团队，可迁移的部分要按成本排序。最便宜的一条是把 train-rollout discrepancy 提为一等指标：不需要改任何 QAT 逻辑就能测出当前 FP8 rollout 是否在漂，这是本工作最有杠杆的副产品。其次是配置纪律：让训练侧 fake-quant 与 rollout 共享 block scale 布局、共享舍入语义，成本近零，收益是把架构漂移从重要性比率里移除。再次是精度折中：若承担不起 FP4 QAT 的训练改造，FP8 KV + BF16 权重 rollout 能拿到大部分显存收益，同时把 router 扰动控制在 top-k 边界之外。最后，若真要设计指导协议，先在自己的栈上重测"99% 的不一致只跨一个码点"这条统计——它是紧凑缓存方案的全部前提，且强依赖架构、block size 与 kernel 实现，不能直接沿用。 ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]

## 关联

- [[concepts/inference-optimization|推理优化]] — FP4 Rollout 与 KV Cache 低精度化的吞吐账
- [[concepts/moe-mixture-of-experts-2025|MoE]] — router 分数对数值扰动的敏感性
- [[concepts/reinforcement-fine-tuning-rft|RFT]] / [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] — TRACE 作用于 RL rollout 侧
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]]
- [[moc/reinforcement-learning-rlhf|RLHF 主题地图]] / [[queries/llm-training-rl-research|LLM 训练与 RL 研究导航]]

→ [[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767|原文存档]] ^[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767.md]
