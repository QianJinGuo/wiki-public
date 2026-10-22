---
title: "PyTorch 2.15 MPS 后端：FP8、MoE 解码与 Metal Kernel 迁移"
created: 2026-10-20
updated: 2026-10-22
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

## 深度分析

### 算术强度悬崖：decode 该按带宽设计

4096×4096 的 BF16 线性层，单行输入要把约 33.6 MB 权重整块读进来，却只做 3355 万次浮点运算——算术强度恰好 1 FLOP/Byte；统一内存带宽 800 GB/s、BF16 算力数十 TFLOPS，roofline 拐点落在 30–70 FLOP/Byte，于是 decode 的 GEMM 只有峰值的 2%–6% 在跑，其余时间都在等内存。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

三条推论。收益上限由「读多少字节」定而非 tile 调优——大层与 LM head 卡在 1×，MoE gate 这类小投影却吃到超额收益，正因它原本连带宽都没喂满。批处理是唯一翻盘点：batch=B 把权重读摊到 B 个 token，算术强度近似乘 B，B 到 32 上下重回算力受限，故 Mac 上做服务化推理，连续批处理远比单请求调 kernel 重要。context 一长 KV cache 会盖过权重：32 层 8 KV head、head dim 128 的模型每 token 的 KV 约 128 KiB，4K 上下文单序列每步读约 537 MB，已与一整层权重相当——2.16 让 attention 全上 Metal 治的正是这个新瓶颈。

### FP8 在 MPS 上：收益面与三个陷阱

收益分两层：**容量**上 `float8_e4m3fn` 一字节一值、权重减半，对 35B 级 MoE 进统一内存是决定性的；**带宽**上既然 decode 受限在内存，少读一半理论上有近 2× 空间。后者兑现有三个前提：

- **单行路径仍在**——`scaled_mm` 在 batch=1 时须仍走 GEMV 而非退化成通用 GEMM，必须实测。
- **e4m3 范围窄**（最大 448，最小正规数约 2^-6）。per-tensor scale 撞上少数离群通道，会把其余通道压进次正规区甚至归零，而真实权重几乎总有离群值——**rowwise（按输出通道）该是默认**。
- **非 GEMM 别碰**。LayerNorm、RoPE、softmax、embedding、残差要么精度极敏感，要么本就不省带宽，累加与非线性仍应留在 BF16/FP32。

### Metal 迁移的真实成本

CUDA → Metal 是把并行分解重做一遍：threadgroup memory 只有 32 KB 量级，BF16 下一个 64×64 tile 就吃掉大半预算，tile 尺寸、寄存器压力、边界 mask 得一起解，而真实 shape 很少是 tile 整数倍，掩码写错就是静默越界；SIMD 宽度 32 lane，跨 simdgroup 归约要经 threadgroup memory 中转，softmax/reduction/top-k 的加速几乎全压在这个细节；GPU 与 CPU 争同一条总线，带宽饱和后 kernel 再满也无额外收益，故第一版该追「打满带宽」而非「打满 ALU」。

不该 port 的：调用频率低、开销由 dispatch 主导（GPU 省了，CPU 编码反成主项）；形状组合爆炸而 MPSGraph 已够快；占比 1% 的尾部。该 port 的是高频、带宽受限、MPSGraph 会物化中间张量的路径——8,764 次 padding 与 128k 词表 softmax 治的是同一个病。

### 静默正确性：唯一不可接受的失败模式

Qwen-Image-2.1 那例的可怕不在 10.06 dB，而在它**不报错**：图能出、流程跑完、日志干净，只有人眼看出不对。崩溃是廉价的（栈回溯直指缺失算子），静默错误则把故障转移到下游，让用户以为问题在模型或提示词。^[raw/articles/pytorch-215-mps-backend-optimization-2026.md]

验证因此要前置：生成类用任务级指标（PSNR/SSIM、往返一致性），判别类按 dtype 设容差（bf16 取 1e-2 量级 rtol、fp32 取 1e-5），而非一个 `allclose` 拍所有情况；算子级逐层对比最省时间，同权重同输入在 CPU 与 MPS 各跑一遍逐 op 比最大误差；形状边界必须专测——padding、反射填充、非 8 倍数长度、batch=1 与 N。

### MPS + FP8 与 GGUF 的分工

两条路治同一个病（带宽），药方不同。GGUF 把每个字节压到极致：Q4_K_M 这类 4 bit 每 token 只读 FP8 一半的权重，纯 decode 吞吐完全可能跑赢 FP8，且跨厂商可移植、CPU/GPU 混合 offload、内存不足时的降级路径都成熟。MPS + FP8 赢在别处：留在 PyTorch 里（训练、LoRA、自定义算子、`torch.compile`、新架构 day-one，不必过格式转换）；大模型整块驻留统一内存；精度预算比 4 bit 宽，长上下文与需要稳定结构化输出时更安全。它输在只成立于 macOS、量化生态不如 GGUF 成熟、算子覆盖仍有洞。选型三条：改模型或训练 → MPS；只跑且内存吃紧 → GGUF；两者都要 → GGUF 做基线、MPS 做实验。可对照 [[entities/quantization-techniques|量化技术]]、[[concepts/moe-mixture-of-experts-2025|MoE 架构]]。

## 实践启示

1. **先量带宽，再调 kernel。** 用 `[1, 1, hidden]` 的真实 decode shape 跑 profile，确认耗时在内存读而非 dispatch；若 KV 读取已超过权重（4K 上下文、32 层以上常见），重点转向 KV 量化/分页与 attention。
2. **升 2.15 先做正确性回归，再比性能。** 固定 prompt 集与 seed，CPU 生成 golden 逐 op 比对；有 VAE、卷积、padding 的 diffusers 流程另加 PSNR/SSIM 往返检查：2.15 修好了 2.14 的静默损坏，同批改动也可能改别的边界。
3. **FP8 只上大 GEMM 权重、scale 用 rowwise。** 归一化、RoPE、softmax、embedding、残差保持 BF16；`output_dtype` 设 bf16、累加不下探；对比 batch=1 与 32 确认未从 GEMV 退化。
4. **MoE 先确认 group offset 不再回 CPU。** 用 `F.grouped_mm` 加 `torch.compile` 查有无 device-host 同步；小 group kernel 是否命中决定 5–6.5× 能否拿到，top-k routing 与之同属关键路径。
5. **用自己的 model zoo 验证「1.23× 是几何均值」。** 收益集中在小投影，大层与 LM head 接近 1×；以超大 dense 层为主的模型别指望端到端同等提升。
6. **按「是否值得 port」而非「能否用 Metal 写」决定动手。** 判据是调用频率 × 单次数据量，且 MPSGraph 版本会物化中间张量；一次性、形状奇特的算子留给上游。

## 关联

- [[concepts/inference-optimization|推理优化]] — decode 期带宽瓶颈与 kernel 融合
- [[entities/llama-cpp-deployment|llama.cpp 部署]] — 另一条本地推理路径（GGUF/量化）
- [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|端侧 agentic 小模型]] — 端侧部署的另一端

→ [[raw/articles/pytorch-215-mps-backend-optimization-2026|原文存档]]
