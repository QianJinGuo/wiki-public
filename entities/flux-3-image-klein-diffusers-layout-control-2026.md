---
title: "FLUX.3 Image Klein 接入 Diffusers：布局控制与 day-one 集成"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [image-generation, diffusion, flux, diffusers, layout-control, lora, quantization]
sources: [raw/articles/flux-3-image-klein-diffusers-layout-control-2026]
confidence: 0.8
---

# FLUX.3 Image Klein 接入 Diffusers：布局控制与 day-one 集成

Black Forest Labs 的 FLUX.3 Image Klein 在发布首日即进入 Diffusers：**一个模型同时做文生图、多参考图生图和包围盒布局控制**——你可以把 bounding box 写进提示词，指定每个元素应该落在画面哪里，或指定参考图中哪一部分被移动。这是 [[entities/flux-3-multimodal-flow-model-black-forest-labs-2026|FLUX.3 多模态流模型]] 家族在开源侧的第一个「可控布局」变体。^[raw/articles/flux-3-image-klein-diffusers-layout-control-2026.md]

## 两个 checkpoint

| Checkpoint | 定位 | 默认参数 |
|---|---|---|
| `black-forest-labs/flux-3-image-klein` | 时间步蒸馏版，追求快 | 16 步、无 CFG |
| `black-forest-labs/flux-3-image-klein-base` | 基座模型 | 50 步、带 CFG |

蒸馏版在 H100 上生成 1024×1024 约 **2 秒**；NVFP4 与 FP8 权重可通过 `from_single_file` 直接加载，供显存较小的 GPU 使用；官方提供文生图与图像编辑两套 LoRA 训练脚本。^[raw/articles/flux-3-image-klein-diffusers-layout-control-2026.md]

## 用法：ModularPipeline

```python
import torch
from diffusers import ModularPipeline

pipe = ModularPipeline.from_pretrained("black-forest-labs/flux-3-image-klein")
pipe.load_components(dtype=torch.bfloat16)
pipe.to("cuda")
pipe(prompt="a red fox", output="images")[0].save("fox.png")
```

需要最新的 `diffusers` 与 `transformers`（文本编码器为 Qwen3-VL）。模型以 **Modular Diffusers** pipeline 形式发布，可以按需拆出模块或插入自定义块——这条设计与 [[concepts/inference-optimization|推理优化]] 的「可组合管线」思路一致：先让结构透明，再做针对性加速。^[raw/articles/flux-3-image-klein-diffusers-layout-control-2026.md]

## 模型结构：三类 token

模型内部接收三种 token：**文本**、**正在生成的噪声图像**、**可选参考图像**。多参考图生图与布局控制都建立在这套统一 token 化之上，因此「编辑」与「生成」不是两条独立流水线，而是同一去噪过程的不同输入组合。^[raw/articles/flux-3-image-klein-diffusers-layout-control-2026.md]

## 值得注意的三点

1. **布局控制是提示词级的**：bounding box 写在文本提示里，不需要额外的布局编码器或控制网（ControlNet 式外挂）。这意味着控制信号与语义在同一条条件通路上竞争，泛化性好但精度上限取决于文本编码器对坐标的理解。
2. **量化权重在发布即提供**：NVFP4/FP8 与蒸馏版组合，使 2 秒级 1024×1024 生成可以在消费级/中端卡上落地，与 [[entities/nunchaku-4bit-diffusion-inference-diffusers|Nunchaku 4-bit 扩散推理]] 走的是同一条「低比特 + Diffusers 原生」路线。
3. **LoRA 训练脚本齐备**：文生图与图像编辑各有脚本，说明官方把「二次微调」当作一等公民而非社区补丁。

## 局限

官方博客以能力展示与用法为主，未给出布局控制的定量指标（box 对齐精度、多参考图身份保持度）；蒸馏版 16 步的效果上限与基座 50 步的差距只有定性描述。

## 关联

- [[entities/flux-3-multimodal-flow-model-black-forest-labs-2026|FLUX.3 多模态流模型（BFL 官方发布）]]
- [[entities/nunchaku-4bit-diffusion-inference-diffusers|Nunchaku 4-bit 扩散推理]]
- [[concepts/inference-optimization|推理优化]]

→ [[raw/articles/flux-3-image-klein-diffusers-layout-control-2026|原文存档]]
