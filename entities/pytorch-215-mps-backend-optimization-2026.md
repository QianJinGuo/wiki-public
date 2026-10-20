---
title: "PyTorch 2.15 MPS 后端：FP8、MoE 解码与 Metal Kernel 迁移"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [pytorch, mps, apple-silicon, inference, moe, fp8, metal]
sources: [raw/articles/pytorch-215-mps-backend-optimization-2026]
confidence: 0.85
---

# PyTorch 2.15 MPS 后端：FP8、MoE 解码与 Metal Kernel 迁移

## 一句话定位

Hugging Face 的 PyTorch 团队在 PyTorch 2.15 里系统性补齐 Apple Silicon 的 MPS 后端：FP8 首次落到 Mac、MoE 解码 5–6.5× 加速、视频 VAE 解码显存峰值从 270 GiB 降到 8 GiB，路径主线是**用专门的 Metal kernel 替换 MPSGraph**。这与 [[concepts/inference-optimization|推理优化]] 里「瓶颈常不在算力而在带宽与 kernel 边界」的判断完全吻合。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## 关键收益一览

| 维度 | 2.14 | 2.15 |
|---|---|---|
| MoE 解码（M3 Ultra, torch.compile） | Qwen3.6-35B-A3B 9.4 tok/s；LFM2.5-8B-A1B 23.6 tok/s | **61.3 tok/s**；**123.6 tok/s**（5–6.5×） |
| `F.linear` 解码路径 | 走 MPSGraph | 接 GEMV kernel，96 个 BF16 层形状几何平均 **1.23×** |
| sampling softmax（vocab 维度） | 慢 | 约 **8×** |
| Wan2.2 视频 VAE 解码显存 | 270 GiB 峰值 | 约 **8 GiB** |
| Qwen-Image-2.1 VAE 编码往返 | MPS 上静默损坏图像，PSNR 10.06 dB | 修正后 **29.95 dB**（与 CPU 相当） |
| FP8 | 不支持 | `torch.float8_e4m3fn` + `F.scaled_mm`（tensorwise / rowwise 两种 scale） |

^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## Decode 阶段为什么是瓶颈：把 matmul 变成 matvec

文章对 decode 的解释很直接：prefill 是矩阵乘，decode 每步只有一行 token，大 matmul 退化成矩阵-向量积，于是**带宽而非算力**成为主约束。2.15 在两个吃这一步的层上同时动手：

- **dense linear**：MPS 自 2.14 起 `torch.mm` 已有 GEMV kernel，但 `F.linear` 仍走 MPSGraph；2.15 把两者接通，`[1, 1, hidden_size]` 形态的解码输入直接走 GEMV 并融合同类 bias 加法。收益集中在小投影（如 MoE 的 gate），大层与 LM head 本就接近带宽上限，基本维持 1×。
- **MoE 层**：2.14 的 grouped matmul 要把 group offset 拷回 CPU 再逐组循环；2.15 给 MPS 原生实现，Metal kernel 直接在 GPU 读 offset（可用时走 Metal Performance Primitives，否则走另一条 Metal 路径），并为解码期最常见的小 group 单独做了 kernel。

^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## 一条被忽略的工程教训：静默正确性

Qwen-Image-2.1 的 VAE encoder 在 2.14 的 MPS 上会**静默破坏图像**，直接打断 ComfyUI 的图像编辑工作流——崩溃好排查，静默错误才是后端移植里最贵的故障。根因是 padding 行为，同一个 padding 改动（PR #195368）同时把 encode-decode 往返从 10.06 dB 拉到 29.95 dB。这条与 [[entities/llama-cpp-deployment|本地部署]] 场景的经验一致：把模型搬到新后端，第一优先级是数值对齐而不是速度。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## FP8 的用法边界

FP8 张量一字节/值，存储为 BF16 的一半，配合 `F.scaled_mm` 直接在 GPU 上做缩放矩阵乘。文章给的示例是每个张量一个 scale：取 `float8_e4m3fn` 的 max 反算 scale，量化后调用 `scaled_mm`，输出 dtype 指定为 bf16。这意味着 Mac 上第一次有了与 CUDA/CPU 对齐的 FP8 语义，但仍是**权重级量化 + 反量化乘**的形态，不是完整的 FP8 训练栈。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## 其他补齐项

复数 LU 与三角求解、更快的小矩阵求逆与 QR 分解；卷积、padding、dropout、插值反向与 reduction 的加速路径；公开的 MPS stream 与更可靠的事件计时（做多流并行的前提）；以及大量此前不支持的算子。环境要求抬到 macOS 15+ / Python 3.11+，可用性检查会直接说明失败原因。2.16 的方向已预告：attention 全量上 Metal、继续 MPSGraph→Metal 迁移、更快的 LLM 解码。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## 对本地/端侧推理的意义

这类后端改进的价值不在跑分，而在把「Mac 只能跑小模型」推到「MoE 大模型可交互」：Qwen3.6-35B-A3B 在 M3 Ultra 上 61.3 tok/s 已进入可用区间，配合 FP8 与 1/33 的视频解码显存，端侧 agent 的部署边界随之上移。可对照 [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|端侧 agentic 小模型]] 的路线看：一侧压模型规模，一侧压 kernel 与显存开销。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

## 关联

- [[concepts/inference-optimization|推理优化]] — decode 期带宽瓶颈与 kernel 融合
- [[entities/llama-cpp-deployment|llama.cpp 部署]] — 另一条本地推理路径（GGUF/量化）
- [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|端侧 agentic 小模型]] — 端侧部署的另一端

→ [[raw/articles/pytorch-215-mps-backend-optimization-2026|原文存档]]
