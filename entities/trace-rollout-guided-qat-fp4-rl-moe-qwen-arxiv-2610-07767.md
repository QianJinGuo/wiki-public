---
title: "TRACE：Rollout 引导的 FP4 量化感知训练，让 MoE 强化学习又快又稳"
created: 2026-10-24
updated: 2026-10-24
type: entity
tags: [rl, quantization, fp4, moe, post-training, inference-optimization, qwen, arxiv]
sources: [raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767]
confidence: 0.78
provenance_state: extracted
---

# TRACE：Rollout 引导的 FP4 量化感知训练，让 MoE 强化学习又快又稳

阿里千问（Qwen）与俄亥俄州立大学联合提出 TRACE（arXiv:2610.07767），要解决的是强化学习里最贵的一段——Rollout——能不能压到 FP4 还不变崩。结论是能，但前提不是"把两边都量化得更准"，而是**把训练侧与 Rollout 侧的低精度路径差异直接压小**。在 Qwen3.5-35B-A3B 上平均分 75.3（BF16 Rollout 74.9），8K 输入 / 128K 输出时解码吞吐相对 BF16 Rollout 最高 5.4×，端到端 RL step time 只多约 7.4%。

## 问题：FP4 Rollout 为什么把 RL 训练"搞崩"

Rollout 阶段在长轨迹 RL 里占掉大量时间，把 BF16 换成 FP4 能同时省算力和显存，但现有做法（普通 FP4 QAT、QaRL、QUADS）在训练后期普遍出现奖励与测试分数下滑甚至崩溃。TRACE 作者给出的根因不是"量化误差大"，而是**误差度量选错了对象**：

- 已有方法都在降低"量化后的值离原始高精度值有多近"，即分别优化训练路径和 Rollout 路径各自的量化误差；
- 但两条路径各自都量得很准，仍可能被量化到**不同码点**,于是微小差异被放大。论文的例子：训练侧激活 60.24、Rollout 侧 59.76，原始只差 0.48；经同样的 global scale + block scale 归一化后为 2.51 与 2.49，恰好落在舍入边界两侧，一个量化到 3、一个量化到 2，最终差异放大到 24；
- 对 MoE 更致命：数值扰动会传导到 router 分数与专家选择，训练策略与实际生成轨迹对应的策略进一步偏离——这正是 [[concepts/moe-mixture-of-experts-2025|MoE]] 结构下低精度 RL 的放大器。

## 机制一：让训练侧向 Rollout 的量化结果对齐

Rollout-Guided QAT 的核心动作：Rollout 阶段记录 **FP4 routed expert 激活**与 **FP4 KV 状态**对应的量化结果；训练阶段这些信息不只当历史数据，而是直接参与训练侧 fake quantization 的**舍入决策**。

- 训练侧仍只在相邻两个 FP4 E2M1 码点之间选，但不再机械用 round-to-nearest（RTN）；
- 改为优先选择**距离 Rollout 侧实际码点更近**的那个候选值。

这把"两条路径互相靠近"变成了一次显式的舍入方向对齐，而不是两个独立误差最小化问题。

## 机制二：只传"最少必要"的量化信息

完整保存 Rollout 量化结果要经历 GPU→CPU→缓存→磁盘→回 GPU，长轨迹、大 batch、异步 RL 都会放大存储与通信。作者的统计给了捷径：**超过 99% 的训练/Rollout 量化不一致只相差一个相邻 FP4 codebook entry**。既然训练侧自身激活已把候选范围压到极小，Rollout 侧只需保留**少量 mantissa 信息 + scale**，就足以告诉训练侧往哪个方向舍入。实测中若保存完整指导信息，Rollout 会被通信开销严重拖慢；改用紧凑缓存后吞吐贴近原生 FP4 Rollout。

## 机制三（意外发现）：模型会在 RL 中"适应"FP4

训练早期 FP4 执行仍有性能损失，但只要 train-rollout discrepancy 被控制住，模型会逐步适应这个低精度执行环境：

- Qwen3.5-35B-A3B 在 HMMT25 曲线上约 **120 个 RL step** 后追平 BF16；
- Qwen3.8-Flash-Next 上类似"追平"出现在约 **240 step**。

含义是：低精度不必只被当作静态噪声。若训练与生成处在**一致、可学习的低精度环境**中，策略本身会通过 RL 更新适应这种数值约束。这也解释了为什么"先 BF16 训练再事后量化"打不过"训练期就用 FP4"——后者最终 FP4 policy 平均分 75.3，而 Vanilla NVFP4 / 4over6 / H-Scale 事后量化分别是 70.4 / 71.0 / 71.4。

## 数据与边界

| 设置 | 结果 |
|---|---|
| Qwen3.5-35B-A3B 平均分 | TRACE 75.3 / BF16 Rollout 74.9 / QUADS 68.8 / QAT 59.6 / QaRL 58.6 |
| HMMT25 | 从 QUADS 的 59.0 提到 70.0 |
| 解码吞吐（8K in / 128K out） | 相对 BF16 Rollout 最高 5.4× |
| 端到端 RL step 开销 | 约 +7.4% |
| 覆盖模型 | Qwen3.5-35B-A3B、Qwen3.5-122B-A10B、Qwen3.8-Flash-Next、Qwen3.8-2.4T-A95B |

边界要看清：结论建立在 MoE 架构 + FP4 E2M1 码点 + 自家 RL 任务栈上，128K 输出下的吞吐优势并不等于所有轨迹长度都同样受益；"模型适应低精度"目前是曲线观察，缺少对失败模式（何时不追平）的系统刻画。

## 可迁移的三条

1. **一致的低精度环境优于事后量化**：训练与推理用同一套低精度语义，模型会把量化边界当约束学进去，而不是当噪声扛过去。
2. **差异度量和误差度量不是一回事**：两个系统各自误差小，仍可能互相错位；对齐"两条路径的相对位置"往往比叠加两个绝对误差更有效。
3. **传输成本用分布统计砍**：99% 的不一致只跨一个码点，这个统计事实直接决定了紧凑缓存方案可行——先量化"信息需要多精确"，再设计协议。

## 关联

- [[concepts/inference-optimization|推理优化]] — FP4 Rollout 与 KV Cache 低精度化的吞吐账
- [[concepts/moe-mixture-of-experts-2025|MoE]] — router 分数对数值扰动的敏感性
- [[concepts/reinforcement-fine-tuning-rft|RFT]] / [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] — TRACE 作用于 RL rollout 侧
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]]
- [[moc/reinforcement-learning-rlhf|RLHF 主题地图]] / [[queries/llm-training-rl-research|LLM 训练与 RL 研究导航]]

→ [[raw/articles/trace-rollout-guided-qat-fp4-rl-moe-qwen-arxiv-2610-07767|原文存档]]
