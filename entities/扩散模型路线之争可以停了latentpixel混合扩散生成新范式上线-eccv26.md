---
title: "Hi-DiT：混合 Latent-Pixel 扩散 Transformer（ECCV 2026）"
created: 2026-09-28
updated: 2026-09-28
type: entity
tags: [diffusion, image-generation, architecture, transformer, multimodal]
sources: [raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26]
confidence: 0.75
provenance_state: extracted
---

# Hi-DiT：混合 Latent-Pixel 扩散 Transformer（ECCV 2026）

> 中科大 + 智象 HiDream.ai 提出的 Hybrid Latent-Pixel Diffusion Transformer，围绕扩散去噪的时间异质性重新分配模型能力，论文被 ECCV 2026 接收。

## 问题：两条路线的固有矛盾

高分辨率图像生成的两条主要路线各有瓶颈：**Latent diffusion** 经 VAE 压缩高效稳定，但压缩/解码丢失高频信息，纹理、边界、局部细节恢复有上限；**Pixel diffusion** 直接在像素或 patchified Pixel tokens 上建模理论上更保细节，但原始像素分布维度高、熵大，模型需在同一去噪过程中同时学全局语义与局部高频纹理，易出现容量竞争、优化缓慢、训练不稳定。^[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26.md]

## 核心洞察：去噪过程时间异质

扩散去噪并不是「每个时间步都同质」的过程：高噪声早期偏向恢复低频结构与全局语义，低噪声后期才需要合成边缘纹理等高频细节。与其让同一个模型全程同时承担两种目标，不如让 Latent 与 Pixel 在不同阶段各司其职——早期用 Latent stream 做全局规划（semantic scaffold），后期打开 Pixel stream 补高频细节。^[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26.md]

## 两个关键设计

- **Time-Gated Injection**：门控函数 G(t)=1(t<τ) 控制 Pixel tokens 注入时机，高噪声阶段关闭 Pixel stream 避免过早承担局部纹理预测，默认 τ=0.3。
- **High-Frequency Pixel Predictor**：传统 Pixel diffusion 用单层线性投影从 token 直接回归大尺寸 patch 回归难度高；Hi-DiT 用分层 sub-Pixel decoding——先卷积 + PixelShuffle 逐步上采样，再预测局部 sub-patch，降低高维像素回归难度。

模型整体运行在参数共享的 Transformer backbone 内，Latent 与 Pixel tokens 可交互，通过不同输入投影和输出预测头承担不同目标。^[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26.md]

## 实验结果

- ImageNet 256×256 CFG 设置达 **1.06 FID**；ImageNet 512×512 达 **1.26 gFID**；MS-COCO 文生图语义对齐更紧。
- 消融（80 epoch 快速验证）：vanilla Pixel stream 无 Latent 输入 FID=3.38 → 加 Latent input 降至 1.74（Latent 提供语义支架）→ 加 High-Frequency Pixel Predictor 达 1.65。
- 时间门控阈值消融：τ=0.7（Pixel 打开太早）重新引入容量竞争；τ=0.1（太晚）细节恢复不足；τ=0.3 最优 FID=1.65 / IS=267.7。
- 效率：对比 SiT/VA-VAE，每 epoch 训练 17.1/17.7 → 18.1 分钟，单图推理 1.52/1.58 → 1.67 秒，峰值显存 35.52G——小幅开销换 FID 2.06/1.35 → 1.06。^[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26.md]

## 洞察

Hi-DiT 的贡献不是简单「Latent+Pixel」拼接，而是**让不同表征在去噪轨迹的不同阶段发挥不同作用**的时间感知容量分配。这为图像生成基础模型给出了明确方向：不必在两条路线间二选一。与 [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD 自回归扩散]]、[[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model|CoLa-DLM 连续 latent 扩散语言模型]] 同属「Latent/Pixel/连续表征混合」的架构创新谱系，但 Hi-DiT 的独特性在于把混合维度放在时间轴（噪声阶段）而非空间或模态轴。代码开源：github.com/HiDream-ai/Hi-DiT。^[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26.md]

## 关联

- [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD 长视频自回归扩散]] — 扩散架构创新谱系
- [[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model|CoLa-DLM]] — 连续 latent 表征在语言扩散中的应用
- [[entities/acl-2026-diffusion-lm-block-size-reasoning-t-star|Diffusion LM block-size 推理]] — 扩散语言模型推理控制

→ [[raw/articles/扩散模型路线之争可以停了latentpixel混合扩散生成新范式上线-eccv26|原文存档]]
