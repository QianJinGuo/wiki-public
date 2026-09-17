---

title: "【多模态理解与生成统一模型】LLM+flow模型生成范式原理与代码解析"
type: entity
created: 2026-07-04
updated: 2026-09-14
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 【多模态理解与生成统一模型】LLM+flow模型生成范式原理与代码解析

## 摘要

这篇文章把「既能看图、又能画图」的多模态统一模型拆成两条路线：LLM + 离散图像 token（Chameleon、Janus、show-o）与 LLM + 连续图像生成器（Transfusion、JanusFlow）。作者以 DeepSeek 的 JanusFlow 为主线，讲清它如何在只共享 LLM 主干的前提下，让理解走 SigLIP、让生成走 ConvNeXt 堆叠的 rectified flow 编解码器。文章给出推理代码的张量形状、时间步嵌入与 CFG 的实现细节，以及三阶段训练流程和两项生成 loss 的形式。^[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析.md]

## 核心要点

- 两大范式：LLM + 离散图像 token（VQ 码本，自回归或类 BERT 完形填空），与 LLM + 连续图像生成器（扩散或 rectified flow）。
- 第一类范式代表 Chameleon、Janus（自回归出 image token）与 show-o（mask 预测并行产出）；第二类代表 Transfusion（扩散）与 JanusFlow（rectified flow）。
- JanusFlow 的理解与生成不共享图像编码器：理解用 SigLIP，生成用 ConvNeXt block 堆叠的 ShallowUViT 编解码器，只共享 LLM 主干。
- 生成发生在 latent space：噪声为 [bz,4,48,48]，flow decoder 输出同形状速度场，再由冻结的 SDXL-VAE decoder 还原到像素空间 [3,384,384]。
- flow encoder 把噪声变换成 [768,24,24] 的 feature map，并把时间步 t 编码为 2048 维 t_emb，让图像通路感知当前迭代进度。
- 生成对 LLM 只前向一次（类 prefill），但 flow ODE 需迭代约 30 步，每步核心更新只有 z ← z + dt·v。
- 训练分三阶段：stage1 只训随机初始化组件；stage2 训除视觉理解编码器与 VAE 外的全部模块并动态调整数据配比；stage3 SFT 解冻除 VAE 外所有模块。

## 深度分析

### 两种范式为何分野：离散 token 与连续生成器

分叉点在于「图像被表示成什么」。第一类范式用 VQ-GAN/VQ-VAE 训出一张离散码表，把图像压成若干 token id，生成于是退化为 LLM 最擅长的 next-token prediction：模型自回归吐 token，或像 show-o 那样一次并行补全多个被 mask 的 image token，再解码回像素。其优势是复用 LLM 的全部训练基础设施与采样技巧，代价是量化带来的信息损失与长序列推理成本。第二类范式放弃离散化，让 LLM 与连续生成器紧密耦合：LLM 负责语言理解与条件建模，生成器负责在连续空间产出像素。分工看似优雅，却意味着理解与生成无法共用同一套视觉表示，模型内部天然存在两条视觉通路。^[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析.md]

### JanusFlow 的架构切面：理解与生成在哪里分叉

JanusFlow 的答案是把两个任务的图像侧彻底拆开、只在 LLM 主干处汇合。理解时图像经 SigLIP 编码，走 Qwen-VL/DeepSeek-VL 那套视觉—文本对齐流程。生成时输入不是图像而是 latent 噪声 [bz,4,48,48]，先由 ShallowUViTEncoder（二维卷积 + ConvNeXt block）变换为 [768,24,24] 的 z_emb 并同步产生 t_emb，再经 768→2048 的 MLP 对齐；随后文本 embedding、t_emb、z_emb 在序列维度直接 concat 喂给 LLM，靠 self-attention 完成文本对图像的条件调制。LLM 输出中最后 576 个 token 承载图像信息，经 dec_aligner 回到 768 维并 reshape 成 [768,24,24]，送入同为 ConvNeXt 堆叠的 ShallowUViTDecoder，用 PixelShuffle 完成 24×24→48×48 上采样。共享面仅限 LLM 主干，视觉编码器与解码器均任务专属。^[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析.md]

### latent space、rectified flow 与扩散的取舍

在 latent space 而非像素空间生成，是效率前提：48×48×4 个位置远少于 384×384×3 的像素量，编解码器与 LLM 的序列长度都因此可控。至于为何选 rectified flow 而非更主流的扩散，作者给出人事与工程两重解释：rectified flow 作者也在 DeepSeek，且是 JanusFlow 二作；技术上则因位移—速度抽象极简——训练时假设从噪声 Z0 到目标 Z1 走直线，因此不同 t 的回归目标都是同一个 Z1−Z0，采样时反复执行 z ← z + dt·v。相比扩散动辄数十上百步的噪声调度，直线假设让采样路径更短，同时保留了扩散生态的实现范式（正弦时间编码、TimestepEmbedding 均复用 diffusers）。^[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析.md]

### 训练配方、数据配比与工程权衡

训练同样三阶段渐进：stage1 冻结预训练主干，只训随机初始化的适配器与 flow 编解码器；stage2 解冻除视觉理解编码器和 VAE 外的模块，前期多用理解数据、后期加大生成数据（以 data ratio 控制理解 : 生成 : 纯文本比例）；stage3 SFT 学习指令遵循。损失由三项构成：理解的交叉熵、生成主 loss（对全部 t 回归 Z1−Z0），以及对齐辅助 loss——把 LLM 中间表示经 MLP 对齐到图像理解编码器维度，计算时对理解编码器停梯度，以防生成任务把主干表征拉离理解能力。这暴露了统一模型的真实代价：共享参数越多，任务间拉扯越强，只能靠辅助 loss 与数据配比稳定。作者实测也相当诚实——同一 prompt 下 JanusFlow 的生成质量（尤其中文 prompt）反而比 Janus 更糊，说明「统一」目前仍是工程折中，而非免费的协同增益。^[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析.md]

## 实践启示

1. 先定图像表示与生成器：已有成熟 VQ 码本与 LLM 训练栈时，离散 token 范式复用成本最低；若生成质量是硬指标，优先选连续生成器，且 rectified flow 相比扩散采样步数更少、可直接复用 diffusers 组件，迁移成本极低。
2. 评估统一模型必须分开压测理解与生成两条链路。共享主干不等于质量同步提升，理解不掉分不代表生成不糊。
3. 生成侧优先在 latent space 建模，并固定一个预训练 VAE（如 SDXL-VAE）不参与训练；这会显著降显存与不稳定因素，但 latent 分辨率直接决定生成上限。
4. 时间步必须以 embedding 形式注入生成编解码器（t_emb 经条件 scale/shift 调制 ConvNeXt block），否则模型无法感知迭代进度，ODE 难以收敛。
5. CFG 需在 batch 维度成对复制 prompt（一半用 pad 填充做无条件分支），再按 batch 劈开加权求和；这是指令跟随度与画质/多样性的调节旋钮。
6. 训练时加入「LLM 中间表示对齐到理解编码器」的辅助 loss（对理解编码器停梯度），是缓解多任务表征漂移的低成本手段，值得自研统一模型复用。

## 相关实体

- [[entities/vlm详解视觉语言模型原理及代码以deepseek-vl为例|VLM 原理与 DeepSeek-VL 代码解析]]
- [[entities/omnishow-unified-multimodal-video-generation-icml-2026|OmniShow：统一多模态视频生成]]
- [[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model|Cola-DLM：连续潜空间扩散语言模型]]
- [[entities/llava-onevision-2-full-frame-rate-vlm|LLaVA-OneVision-2 全帧率 VLM]]
- [[entities/deepseek-vision-primitives|DeepSeek 视觉原语]]

→ [[raw/articles/多模态理解与生成统一模型llmflow模型生成范式原理与代码解析|原文存档]]
