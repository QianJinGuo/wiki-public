---
title: "OmniShow：极简干预统一多模态可控视频生成（ICML 2026）"
created: 2026-08-21
updated: 2026-09-07
type: entity
tags: [video-generation, multimodal, diffusion, paperweekly, icml-2026, architecture, generalist]
sources: [raw/articles/omnishow-unified-multimodal-video-generation-icml-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OmniShow：极简干预统一多模态可控视频生成（ICML 2026）

> **Background**：PaperWeekly 对 ICML 2026 论文《OmniShow: Unifying Multimodal Conditions for Human-Object Interaction Video Generation》（arXiv:2604.11804，港中文/字节/莫纳什/港大）的解读。OmniShow 用一个 12.3B 模型统一 text、reference images、audio、pose 多模态可控视频生成，核心是"极简干预"（Minimalist Intervention）与"从专才到通才"（Specialists → Generalist）两条设计哲学。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 任务难点：不是支持，而是协同

text prompt 负责全局语义与场景，reference images 锚定人物身份与物体外观，audio 驱动嘴型/表情/身体节奏，pose 给出逐帧精确动作。单独支持任一项不稀奇，难点在于时序、空间、语义三层面彼此不打架——人物不能换脸、商品不能变形、嘴型咬住声音、肢体服从 pose、整体忠于文本。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

现有路线大多偏科：R2V 擅长 reference 但缺音频响应；A2V 能被声音驱动但常只支持首帧；pose-guided 方法能控动作却在身份保持与音画同步上不完整；部分方案还要额外索取 mask/trajectory/depth/bounding box。把子系统硬级联既笨重又易在交界处崩坏。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 哲学一：极简干预，让视觉条件"顺势"进来

只要真正理解 base model 的原生输入结构与学习动态，极小的改动也能撬动强多模态控制。OmniShow 复用 base model（Waver 1.0，12B MMDiT）原生的 channel-concat 机制——reference images 与 pose 都是视觉信号，不另设分支。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

- pose 渲染成 RGB 视频过 VAE 编码；reference images 经 VAE 编码后在 temporal 维增设 pseudo-frame tokens 承载，pose 与 noisy video tokens 对齐，task adaptation gap 极小。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- 补一道极轻的 Reference Reconstruction Loss：pseudo-frame tokens 用同 timestep 加噪后的 reference image tokens 初始化，要求重建语义细节，把"保真"写成显式优化目标。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

### 被"分析"出来的音频架构

声音是连续、含节奏与局部时间关系的模态，硬塞 channel 不合适。OmniShow 设计 Gated Local-Context Attention：Wav2Vec 2.0 融合多层特征，sliding window（window=5, stride=4）对齐视频 fps，masked attention 让每个 latent frame 只 attend 对应局部 audio tokens。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

- Adaptive Gating：learnable gating vector 初始化为 near-zero，让音频影响从极弱处缓慢生长，不扰乱 pretrained feature distribution。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- gate norm 反向成为"诊断工具"，据此判断只在 dual-stream blocks 插入。最终音频模块仅增约 2.5%，规模达 12.3B——架构是被分析出来的，不是被堆出来的。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 哲学二：从专才到通才，把异构数据用到极致

完整多模态样本（同时满足 text/reference/audio/pose 与目标视频质量）极其稀缺。OmniShow 构建多层异构数据流程，把 R2V/A2V/RA2V/RAP2V 各类数据纳入：从 human-centric 视频池出发，shot segmentation 切分，再按分辨率/美学/运动强度/OCR 逐层过滤。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

- Decoupled-Then-Joint Training：先分别训练 R2V 与 A2V 两个 specialists，再 weight interpolation 合并（audio modules 继承自 A2V，其余按 A2V/R2V=0.6/0.4 融合），随后在 RA2V 上继续训练、高质量子集提纯，pose 最后引入避免过早依赖强监督。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- 关键发现：还没显式训练 RA2V，合并后模型即涌现出 zero-shot joint reference-audio 生成能力——"可控性可经由 weight merging 涌现"是"从专才到通才"最直接证据。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 评测：HOIVG-Bench 与多设置结果

构建 HOIVG-Bench（135 个精选样本，配齐 detailed caption、人物/物体 reference、语义对齐 audio、coherent pose），从 Text Alignment / Reference Consistency / Pose Accuracy / Audio-Visual Synchronization / Video Quality 五维评测。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

- R2V 设置：NexusScore 0.389 居首，FaceSim 0.874 紧追 Phantom-14B（0.876），AES 0.468、VQ 11.12、MQ 5.885 三项第一。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- RA2V 设置：Sync-C 8.612、Sync-D 7.608 领先 HuMo-17B（8.013/8.316），音频加入后音画同步/一致性/画质同时提升而非此消彼长。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- RP2V 设置：AKD 0.174、PCK 0.460 优于 VACE（0.206/0.336），NexusScore 0.418、VQ 10.28 领先。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]
- 全部建立在仅 12.3B、音频模块只增约 2.5% 的轻改动之上，对照 HuMo 音频 +21.4%、17B。EMTD benchmark：OmniShow-A2V Sync-C 6.49、AES 1.51（全场最高）、IQA 2.26。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 意义

OmniShow 验证的不是具体技巧，而是一种设计哲学：面对能力越来越强的 base model，稀缺的不是"再加一个新结构"的勇气，而是克制地判断什么该改、什么不该改。先读懂模型原有输入结构，沿熟悉路径接入新条件，再用异构数据与分阶段训练把专才熔成通才。"少即是多"在这里是同时兑现参数效率、训练稳定性、任务统一性与应用可组合性的工程方法论。^[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026.md]

## 相关实体

- [[entities/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607|GenCeption]] — 视频生成模型作为通用视觉学习者
- [[entities/sana-video-2-hybrid-linear-attention-video-generation|Sana Video 2]] — hybrid linear attention 视频生成
- [[entities/tmap-video-generation-inference-acceleration-taobao-2026-07-22|T-Map]] — 视频生成推理加速
- [[entities/flux-3-multimodal-flow-model-black-forest-labs-2026|Flux 3]] — 多模态 flow model

→ [[raw/articles/omnishow-unified-multimodal-video-generation-icml-2026|原文存档]]
