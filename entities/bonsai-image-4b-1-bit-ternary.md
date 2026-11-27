---

title: "Introducing 1-bit and Ternary Bonsai Image Models"
created: 2026-06-02
updated: 2026-11-27
type: entity
tags: [vision, model-compression, 1-bit]
source: [[raw/articles/bonsai-image-4b-1-bit-ternary]]
confidence: 0.75
review_value: 6
sources:
  - raw/articles/bonsai-image-4b-1-bit-ternary
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Introducing 1-bit and Ternary Bonsai Image Models

## 深度分析




![Image 1](https://cdn.prod.website-files.com/699604cc2b9dd89bdbda0608/6a15cec375689f915406cc3c_grid.png)^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


Images generated from Ternary Bonsai Image 4B^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


Today we’re releasing **Bonsai Image 4B**, a family of compact image-generation models designed to run high-quality diffusion inference on local hardware: from laptops to phones. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

Bonsai Image 4B comes in two variants:^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


*   **1-bit Bonsai Image 4B** uses binary {−1, +1} transformer weights with an FP16 group-wise scaling factor, giving 1.125 effective bits per weight. It targets maximum compression and is the right fit when memory pressure, bandwidth, and the deployment footprint are the primary constraints.
*   **Ternary Bonsai Image 4B** uses {−1, 0, +1} transformer weights with an FP16 group-wise scaling factor, giving 1.71 effective bits per weight. The additional zero state gives the model more representational flexibility, improving visual quality and prompt fidelity while remaining extremely compact.

The result is a new deployment regime for image generation: capable outputs, open weights, and practical local inference on devices that were previously out of reach for this class of model. To our knowledge, **Bonsai Image 4B is the first image model in its parameter class to run directly on an iPhone**. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

### 量化设计：二值与三值权重如何构成模型

Bonsai Image 4B 保留 FLUX.2 Klein 4B 的架构不变，只替换扩散 transformer 权重的数值表示：1-bit 变体把权重压成 {−1, +1} 并配 FP16 组缩放因子，等效 1.125 bits/weight；ternary 变体多一个零状态（{−1, 0, +1}），等效 1.71 bits/weight，额外的零保留更多表示灵活性，直接转化为更高的视觉质量与提示保真度。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:21-26] 压缩并非均匀作用于全网：二值层约 14x，但约 5% 精度敏感的投影层仍保留 FP16，因此 1-bit 变体最终 0.93 GB（8.3x）；ternary 层约 10x，最终 1.21 GB（6.4x）——「极端压缩 + 少量高精度支撑张量」的混合结构是整个方案成立的前提。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:48-50]

### 模型体积为何决定本地可用性

扩散 transformer 是 4B 级图像模型中最大、且在每次去噪步骤中反复执行的部分，其体积直接决定内存压力、带宽需求与本地推理速度。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:34-38] 落到数字上：Apple Silicon 部署负载 3.42 GB（1-bit）/ 3.88 GB（ternary），对比全精度 15.97 GB；文本编码器在编码完成后卸载，512x512 平均活跃内存仅 1.5 / 1.96 GB，1024x1024 为 1.95 / 2.38 GB，原模型则分别为 11.74 / 14.39 GB。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:50-52] 部署栈经 MLX 低比特路径（Apple Silicon）与 Gemlite 低比特 GEMM 内核（CUDA）支持 iPhone/iPad/Mac 与 CUDA GPU；在 iPhone 17 Pro Max 上全精度管线放不进内存预算，而两个压缩变体都能上机，512x512 约 9.4 秒出图（Mac M4 Pro 约 6 秒，比全精度 MFLUX 快至 5.6x）。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:54-60]

### 基准方法与数字的实际含义

评估覆盖三个互补维度：GenEval 测物体组合与属性绑定，HPSv3 测人类偏好与美学质量，DPG-Bench 测密集提示遵循与语义忠实。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:62-64] ternary 变体以 1.21 GB 足迹拿到基线 95% 的综合精度（如 GenEval 0.723、DPG 0.851）；1-bit 变体在 1 GB 以下仍保留 88%。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:70-84] 横向对比更能说明问题：面对足迹相近的更小模型（SDXL、BK-SDM-Small、Stable Diffusion 1.5、PixArt-Σ），得分大幅领先，同时与同代 4B 级模型保持竞争力——质量-足迹的 Pareto 前沿被整体推移，现代扩散 transformer 行为进入以往只属于低能力小模型的内存区间。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:75-86]

### 与全精度基线的权衡

两个变体的差异本身就是一组权衡数据：ternary 比 1-bit 多花约 0.3 GB（0.93→1.21 GB），换回约 7 个百分点的精度（88%→95%），零状态的表示收益可观；内存是硬约束时，牺牲这 7 个点可再省约四分之一的 transformer 足迹。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:48-50] 另一半权衡在部署侧：云 API 每个提示都是远程请求，每次迭代都有边际成本与往返延迟，而图像生成天然是迭代活动——改提示、比输出、生成变体、丢弃重试；模型装进设备后，生成嵌入产品体验，更便宜、更快，提示与产物可以不出设备。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:88-97]

## Built for local generation

![Image 2](https://cdn.prod.website-files.com/699604cc2b9dd89bdbda0608/6a15cd893a2c1d8354bece23_ef1081ac.png)^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


Images generated from 1-bit Bonsai Image 4B^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


Local image generation starts with a hard constraint: the model has to fit within the device’s memory budget. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

For a 4B-class image model, the diffusion transformer is the largest part of the model and the part that runs repeatedly during generation. Each denoising step invokes the transformer again, so transformer size directly shapes memory pressure, bandwidth demand, and local inference speed. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

Bonsai Image 4B is built from the FLUX.2 Klein 4B. It keeps the architecture intact but changes how the transformer weights are represented. By moving those weights into binary and ternary form, Bonsai reduces the part of the image pipeline that matters most for local deployment. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

| Model | Diffusion Transformer | Reduction vs FP16 |
| --- | --- | --- |
| FLUX.2 Klein 4B | 7.75 GB | 1.0x |
| 1-bit Bonsai Image 4B | 0.93 GB | 8.3x |
| Ternary Bonsai Image 4B | 1.21 GB | 6.4x |

> **Table I:**Diffusion transformer footprint for models.

The binary layers provide roughly a 14x reduction relative to full-precision transformer weights. A small set of precision-sensitive supporting tensors (~5%), called the projection layers, remains in FP16 so the final 1-bit Bonsai Image 4B transformer is **0.93 GB**: an 8.3x reduction from the 7.75 GB full-precision FLUX.2 Klein 4B. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

The ternary variant follows the same structure. Its ternary layers provide roughly a 10x reduction and the final Ternary Bonsai Image 4B transformer is **1.21 GB**, a 6.4x reduction from the full-precision transformer. It is slightly larger than the 1-bit model, but the additional zero state improves visual quality and prompt fidelity. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

Including the compressed text encoder and FP16 VAE, the Apple Silicon deployment payload is 3.42 GB for 1-bit Bonsai Image 4B and 3.88 GB for Ternary Bonsai Image 4B. For comparison, the full precision FLUX.2 Klein 4B requires a deployment payload of 15.97 GB. Since, at runtime, the text encoder is offloaded after prompt encoding, the mean memory usage is smaller than the total payload. When generating a 512x512 image, the mean-active memory is 1.5 GB and 1.96 GB, for the binary and ternary models, compared to 11.74 GB for the original FLUX.2 Klein 4B (a reduction of 7.8x and 6.0x, respectively). For a 1024x1024 image, the mean-active memory is 1.95 GB and 2.38 GB, for the binary and ternary models, compared to 14.39 GB for the original FLUX.2 Klein 4B (a reduction of 7.4x and 6.0x, respectively). ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

This reduction in memory footprint changes where the model can run. Our deployment stack supports Apple Silicon iPhones, iPads and Macs and CUDA GPUs, using MLX low-bit paths on Apple hardware and Gemlite low-bit GEMM kernels on CUDA. On iPhone 17 Pro Max, the full-precision FLUX.2 Klein 4B pipeline does not fit within the device memory budget, while both Bonsai Image variants run on-device. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

[Video 3](https://vimeo.com/1195512651)^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


> Video I: Image generation on Bonsai Studio

In practice, Bonsai Image 4B generates a 512x512 image in 9.4 seconds on an iPhone 17 Pro Max and about 6 seconds on Mac M4 Pro. On Mac M4 Pro, Bonsai Image 4B is up to 5.6x faster than the stock full-precision MFLUX pipeline. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

## Benchmarking performance

Compression only matters if the model remains useful. We evaluated Bonsai Image 4B across three complementary benchmarks: **GenEval** for object composition and attribute binding; **HPSv3** human preference and aesthetic quality; **DPG-Bench** dense prompt following and semantic faithfulness. ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]

![Image 3](https://cdn^[raw/articles/bonsai-image-4b-1-bit-ternary.md]


## 实践启示

1. **选型**：内存/带宽硬约束（手机、边缘设备）选 1-bit 变体；能多给约 0.3 GB 且在意质量与提示遵循就选 ternary——以 1.21 GB 的 transformer 足迹保住基线 95% 精度，通常应作默认首选。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:70-84]
2. **硬件与存储**：Apple Silicon 走 MLX 低比特路径，CUDA 走 Gemlite 内核；整包部署 3.42–3.88 GB（全精度 15.97 GB），运行时平均活跃内存仅 1.5–2.4 GB（512–1024px），评估目标设备时看平均活跃内存而非整包大小。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:52-54]
3. **速度预期**：512x512 在 iPhone 17 Pro Max 约 9.4 秒、Mac M4 Pro 约 6 秒，比全精度 MFLUX 快至 5.6x——低比特 GEMM 内核同时带来内存与吞吐双重收益。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:54-60]
4. **评估陷阱**：GenEval/HPSv3/DPG-Bench 分别度量组成、美学与提示遵循，压缩损失非均匀（1-bit −12% vs ternary −5%）；引用这些数字时注意基线是 FLUX.2 Klein 4B，勿与更早的 SD 系混比。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:62-84]
5. **工程取舍**：约 5% 精度敏感的投影层保留 FP16 是量化方案成立的关键；自行复现类似量化时，不要把精度敏感张量一并压到极端位宽。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:48-50]
6. **场景判断**：高频迭代、隐私敏感（提示与产物不出设备）的生成场景最能受益；低频一次性调用仍可走云 API——本地化的核心收益是迭代成本与往返延迟趋近于零。^[raw/articles/bonsai-image-4b-1-bit-ternary.md:88-97]

## 相关实体
- [[entities/bonsai-image-4b-quantization]]
- [[entities/gemma-4-qat-models-optimizing-compression]]
- [[entities/stochastic-parrot-language-models-and-meaning]]
- [[entities/openai-models-codex-amazon-bedrock-ga]]
- [[entities/stochastic-parrot-language-models-and-meaning.md]]
- [[moc/vision-multimodal|MOC]]

→ [[raw/articles/bonsai-image-4b-1-bit-ternary|原文存档]] ^[raw/articles/bonsai-image-4b-1-bit-ternary.md]
