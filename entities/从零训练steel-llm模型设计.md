---

title: "【从零训练Steel-LLM】模型设计"
type: entity
created: 2026-07-04
updated: 2026-09-20
tags: [wechat, ai]
rating: v7c7
sources:
  - raw/articles/从零训练steel-llm模型设计
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 【从零训练Steel-LLM】模型设计

**来源**: 炼钢AI（作者知乎「战士金」） · **发布日期**: 2024-09-03

**原文链接**: https://mp.weixin.qq.com/s/JaZyf1jOEOtNDCcFqSj8TQ

---

## 摘要

这是「从零训练 Steel-LLM」系列的第三篇（2024-07-09 首发于知乎），讲模型结构设计上的取舍。目标是从 0 预训练约 1B 的中文 LLM，最终训练 1060k step、消费 1.1T token（2 个 epoch），中间 checkpoint 已上传 HuggingFace；改造集中在 FFN 层（SoftMoE 加双层 SENet），attention 原地不动。文中披露了三版 SoftMoE 的实现与失败原因、两处数值精度细节，以及 scaling law 反算结果与工程现实的偏差。^[raw/articles/从零训练steel-llm模型设计.md]

## 核心要点

- 项目定调 1B 规模 + T 级数据；最终 1060k step / 1.1T token / 2 epoch，代码与 checkpoint 已开源（`zhanshijinwat/Steel-LLM`、HF `gqszhanshijin/Steel-LLM`）。
- scaling law 未用 Chinchilla 拟合参数，而用 DeepSeek 技术报告的参数；单卡 A100 实测约 1.88×10^14 flops/s，设训练 25 天时算出「最优」约为 10B 模型 / 单卡 36B 数据。
- 作者自己推翻该结论：10B 模型 MFU 会下降、实际 flops 到不了实测值，单卡也放不下——于是明确「不严格遵守 scaling law，尽量多消费数据」。
- 结构上只改 FFN，self-attention 沿用原生实现 + Flash Attention V2（自造结构就得自写高效算子）。
- 选 SoftMoE 而非 hard MoE 的理由是显存：hard MoE 省计算不省显存，效果比较只在「相同激活参数量」层面成立。
- 三版 SoftMoE：V1 慢且 batch=2 即 OOM；V2 效率高但 loss 收敛不到正常水平、原因不明；V3 效率与显存达标，最终采用。
- 双层 SENet：把 ImageNet 2017 冠军方案的通道注意力简化到 W=H=1 的一维形式，让模型自学神经元增益/衰减；最终 FFN 为 6 个 expert 的 SoftMoE，每个都是 SteelSENet。

## 深度分析

### 架构改造的边界：为什么 self-attention 原地不动

Transformer 可动的地方主要是 self-attention 与 FFN。作者在 attention 上选择「不动」：Flash Attention V2 在序列长度 2048、batch size 2 的对比中，相对 PyTorch 原生实现在训练速度和显存占用上同时占优；而改造 attention 结构就必须配套实现高效算子，作者自认没这方面经验与精力，于是沿用最原始的 self-attention + Flash Attention V2，工程上直接调 PyTorch 的 `scaled_dot_product_attention`，注意 PyTorch 必须升到 2.2 以上，否则内置的仍是 FA V1。^[raw/articles/从零训练steel-llm模型设计.md]

### SoftMoE：hard MoE 的显存悖论与三版实现迭代

MoE 起源于 1991 年的《Adaptive Mixtures of Local Experts》，在 LLM 里落在 FFN 层且普遍是 hard MoE——每个 token 只由部分 expert 计算，动机是省算力。但作者点出常被忽略的代价：**hard MoE 并不怎么省显存**，因为训练与推理仍要把完整模型加载进显存；因此带 hard MoE 的模型对比默认只在「相同激活参数量」层面进行。例：Qwen1.5-MoE-A2.7B 声称与 Qwen1.5-7B 相当，但它整体约有 14B 参数，激活全部 14B 的模型会明显更强。^[raw/articles/从零训练steel-llm模型设计.md]

Steel-LLM 显存有限、希望每份参数都真正参与计算，又要保住 MoE 结构，于是选了 SoftMoE：每个 token 过全部 expert 再按门控权重加权求和（搜广推领域早已成熟）。三版实现中，V1 朴素实现慢且 batch size 2 时显存直接 OOM；V2 复现《From Sparse to Soft Mixtures of Experts》（lucidrains 的 soft-moe-pytorch），效率更高（两次 bmm 先压缩序列维度加权求和再还原），但 loss 无法收敛到正常水平，原因作者至今不清楚；V3 同样复现该论文（bwconrad/soft-moe），效率与显存达标而入选。对比基线是 Qwen1.5-1.8B、8 个 expert、FFN 维度缩 8 倍保持总参数不变。^[raw/articles/从零训练steel-llm模型设计.md]

### 双层 SENet：把 CV 的通道注意力搬到 FFN 第二层

SENet 来自 ImageNet 2017 冠军方案，原用于学习图像各特征通道的重要程度并据此增益或抑制通道。作者做了一次极简降维：假设特征图 W、H 都为 1，三维特征矩阵退化成一根一维向量，SENet 就等价于「输入分别过两个 MLP 得到向量与权重、再逐位相乘」，本质是让模型自己学习对哪些神经元增益、对哪些衰减。顺着这个视角看 Qwen 的 FFN 会发现，其第一层（`gate_proj` 与 `up_proj` 相乘）已与 SENet 思想一致；Steel-LLM 把**第二层**也换成 SENet 形式（`SteelSENet`）。最终 FFN 是 6 个 expert 的 SoftMoE，每个 expert 都是一个 SteelSENet。^[raw/articles/从零训练steel-llm模型设计.md]

### scaling law 算出的「最优」与工程现实之间的偏差

项目起步时就定死「1B 模型 + T 级数据」，但过程中仍按 [[concepts/scaling-laws|scaling law]] 反算了自有算力下「最优」的模型与数据规模。计算未用 Chinchilla 等早期工作的拟合参数，而用 DeepSeek 技术报告给出的参数。wandb 打点显示，训练 1.1B 模型时单卡 A100 实际算力约 1.88×10^14 flops/s（数据并行下各卡独立消费 token，故按单卡口径算）；假设训练 25 天，最低 loss 对应的模型规模约 10B、单卡数据消费量约 36B。^[raw/articles/从零训练steel-llm模型设计.md]

作者随即自己拆掉这个结论：真换成 10B 模型，25 天消费不掉 36B 数据，因为 MFU 会下降、每秒 flops 达不到实测值；何况单卡根本撑不下 10B 训练。更根本的分歧在目标函数：项目要在有限算力下让模型别太小、同时尽量多消费数据学到更多东西，因此并不严格遵守 scaling law。这不是孤例——开源 LLM 的训练数据量普遍远高于公式算出的「最优数据量」，1.1T token 的最终消费正是这种「向上突破公式」的选择。^[raw/articles/从零训练steel-llm模型设计.md]

### 精度细节、checkpoint 策略与「无法自证」的诚实

作者记录了两处精度细节。其一，本想用 flash attention 的 `rms_norm` 算子提一点训练效率，但实测与 PyTorch 实现的 rms norm 无法完全对上——bf16 下逐位存在可见差异，差值虽小仍放弃。其二，SoftMoE 用 softmax 计算 expert 权重，该环节对精度更敏感，故把数值转成 float32 再算（模型以 bf16 混合精度训练）。^[raw/articles/从零训练steel-llm模型设计.md]

文末作者自问「灵魂拷问」：结构上做这么多改造，能证明在 LLM 上有用吗？答案是无法证明——LLM 评估标准太难量化，训练成本又太高，做不起充分消融，这也解释了主流工作为何很少改结构。他的前提是：搜广推出身仍相信结构有收益，模型效果又不影响自己的收入，所以乐意试新想法。兜底手段是 checkpoint——1060k step 无法重跑，多个中间 checkpoint 已上传 HuggingFace，微调与评估正基于它们展开。^[raw/articles/从零训练steel-llm模型设计.md]

## 实践启示

1. 把 scaling law 当预算工具而非教条：先用目标硬件实测 flops 反算「最优」规模，再写下自己真正优化的目标，偏离时连误差来源一起交代。
2. attention 不要轻易自造：无算子工程能力就沿用原生 self-attention + Flash Attention V2，并确认 PyTorch ≥ 2.2。
3. 选 MoE 前先判断瓶颈在算力还是显存：显存吃紧、要让每份参数都参与计算时 SoftMoE 更贴合；对比时必须声明是「激活参数量对齐」还是「总参数量对齐」。
4. 结构改动做口径干净的消融：Qwen1.5-1.8B 基线、8 expert、FFN 维度缩 8 倍保持总参数不变可照抄；并在小步训练里及早发现不收敛的实现。
5. 数值精度别省事：混合精度下 softmax 权重转 float32；与 PyTorch 对不上的融合算子宁可不换，差异在长训练里会被放大。
6. 长期训练靠 checkpoint 兜底：保存并公开中间 checkpoint，既是后续微调/评估的起点，也是「没做消融」时唯一能补的证据。

## 相关实体

- [[entities/从零训练steel-llm微调探索与评估|【从零训练Steel-LLM】微调探索与评估]] — 同系列后续：同一套结构的微调与评估
- [[concepts/moe-mixture-of-experts-2025|MoE（混合专家）]] — hard MoE 与 SoftMoE 的路线背景
- [[concepts/attention-mechanism|Attention 机制]] — 原生 attention 与 Flash Attention V2 的关系
- [[entities/build-llm-from-scratch-7-chapters-zion|从零构建 LLM（七章）]] — 另一条从零训练路线的梳理
- [[entities/fine-tuning-with-trl|TRL 微调]] — checkpoint 之后的微调链

→ [[raw/articles/从零训练steel-llm模型设计|原文存档]]
