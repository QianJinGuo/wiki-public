---
title: "Open d1: Edge Decision Models"
description: "Liquid AI 开源 d1 决策模型家族（d1-3B + d1-omni-600M）：单次前向输出决策而非逐 token 生成，Decision Index v0.2.1 48.57 领先 10B 以下全模型，Jetson 端侧单问 16-50ms，llama.cpp day-one 支持"
created: 2026-10-09
updated: 2026-11-15
type: entity
sources: [raw/articles/liquid-ai-d1-edge-decision-models]
tags: [small-model, edge-deployment, decision-model, liquid-ai, multimodal, inference-optimization, open-weights]
confidence: 0.8
provenance_state: extracted
---

# Open d1: Edge Decision Models

> **Background**：Liquid AI 于 2026-10-07 发布开源 d1 决策模型家族。与生成式 LFM 家族（[[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2.6B]]、[[entities/liquid-ai-lfm2-5-230m|LFM2.5-230M]]、[[entities/liquid-ai-lfm2-5-encoders-fast-long-context-cpu|LFM2.5 Encoders]]）互补，d1 定位"决策"而非"生成"——单次前向直接输出答案，不做 token 序列生成，面向实时结构化决策场景。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 模型家族与核心架构差异

- **d1-3B**：从 LFM2.5-VL-3B（decoder-only VLM）训练，接受文本+图像输入。Decision Index v0.2.1（public split）48.57，领先 10B 以下全部模型，与 12 倍大的 Decider 35B-A3B 持平。训练配方：先平均 LFM2.5-2.6B 与 LFM2.5-VL-3B 文本骨干权重构建更强 base，再以不同种子/数据混合微调多个 checkpoint 后合并；作者强调长输入训练、打乱选项顺序、修数据 shortcut 比更高级技术收益更大。^[raw/articles/liquid-ai-d1-edge-decision-models.md]
- **d1-omni-600M**：首个实验 checkpoint，从双向 encoder LFM2.5-Encoder-350M 训练，覆盖文本+图像与文本+音频两种模态组合。音频路径用 FastConformer encoder + adapter（冻结文本骨干微调音频编码器）；视觉路径从 LFM2.5-VL-450M 取 encoder 训练 adapter + LoRA（仅在图像输入时激活），随后全模型微调、合并 LoRA、与前 checkpoint 权重平均正则化。Decision Index 15.95。^[raw/articles/liquid-ai-d1-edge-decision-models.md]
- **与生成式 LFM 的本质区别**：d1 不产出 token，而是**单次前向传播直接给出答案**。这使延迟计量从 per-token 变为端到端输入→输出，也是端侧实时决策（每帧推理）可行的前提。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 文本 Benchmark（Decision Index v0.3 七项公开基准均值 82.9）

d1-3B 均值 82.9 领先全表（Decider 4B 81.1）；d1-omni-600M 78.4 以 1/4 参数量超 Decider 2B（77.1）。亮点：Civil Comments 毒性检测 95.8（全表最高）、PAWS-X 释义识别 79.5。SQuAD 2.0 85.3 / XNLI 85.0 / BoolQ 87.3。视觉能力沿用 LFM2.5-VL-3B backbone 标准视觉基准验证（专用视觉 split 未公开报告）；音频决策基准被明确指出是 open problem。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 端到端延迟（决策模型的核心指标）

| 设备 | 单问 | 3 问 | 3.4K-token 状态 | 384px 图像 |
|------|-----|------|----------------|-----------|
| Apple M5 Pro | 30ms | 41ms | 640ms | 62ms |
| Jetson AGX Thor | 16ms | 20ms | 220ms | 35ms |
| Jetson AGX Orin 64GB | 26ms | 35ms | 560ms | 83ms |
| Jetson Orin Nano | 50ms | 73ms | 1640ms | 202ms |
| NVIDIA RTX 4090 | 8ms | 21ms | 102ms | 17ms |
| AMD MI325X | 9ms | 14ms | 44ms | 18ms |

单问在所有实测设备 <50ms；3 问仅 1.3× 单问时间（AGX Thor 16→20ms）——单次前向架构下问题数与延迟接近线性且斜率极小。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 部署生态

- 全 NVIDIA 栈（DGX → RTX → Jetson）day-one 支持，llama.cpp 原生支持覆盖 Apple/AMD/Qualcomm/NVIDIA（NVFP4）
- 两个模型均已在 Hugging Face 开放权重，可下载/微调/部署无限制
- 十个实机 demo：d1-3B 对实时摄像头输入逐帧单次前向循环推理（手势控制游戏、实时内容审核等），另与 NVIDIA 合作展示 Jetson + Isaac Sim hardware-in-the-loop 环境导航^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 深度分析

### 单次前向范式重塑延迟经济学

生成式模型的延迟 = 提示词处理 + N×每 token 时间，与输出长度耦合且不可预测；d1 的延迟则是端到端一个常数，只随输入规模变化。这带来两个结构性后果：其一，实时系统可以按帧预算推理——官方十个 demo 全部采用"每帧一次前向"的循环模式（手势游戏、实时内容审核），这在自回归架构下无法保证帧率；其二，多问题共享同一状态的打包推理边际成本极低（3 问仅为单问的 1.3 倍，AGX Thor 16→20ms），叠加原文的"64 states packed"吞吐数据（AGX Thor 262 次/秒、RTX 4090 475 次/秒、MI325X 达 1106 次/秒），单卡即可支撑千次/秒级的高 QPS 决策型负载（审核、路由、意图分类），使"小模型高吞吐"在服务端同样成立，而不仅是端侧卖点。

### 训练配方的反直觉信号：数据卫生胜过高级技术

d1-3B 的配方是"平凡技巧的组合"：先平均 LFM2.5-2.6B 与 LFM2.5-VL-3B 文本骨干权重得到更强 base，再以不同种子/数据混合微调多个 checkpoint 后二次合并。作者明确表示长输入训练、打乱选项顺序、修数据 shortcut 的收益大于更高级的技术——这对选择题式决策训练是个重要警示：模型会学到选项位置等伪相关，数据清洗比方法创新更值钱。d1-omni-600M 则展示了向双向 encoder 增添模态的模板化路径：FastConformer 音频编码器冻结文本骨干单独微调、视觉走 adapter+LoRA（仅图像输入时激活、视觉编码器全程冻结），最后全模型微调、合并 LoRA、与前 checkpoint 权重平均做正则化——分阶段冻结与权重平均贯穿始终，可复用于任何多模态 encoder 扩展。

### Benchmark 结构解读：领先不均匀，边界清晰

d1-3B 的均值领先（82.9 vs Decider 4B 81.1）并非全面碾压：它在阅读理解上优势巨大（SQuAD 2.0 85.3 vs 76.0），但在 BoolQ（86.7 vs 89.0）和 XNLI（85.0 vs 88.6）上反而落后，说明 decoder 血统的 Decider 在自然语言推断类任务上仍有结构性优势。反直觉的是单项最高分（Civil Comments 95.8、PAWS-X 79.5）属于 d1-omni-600M 而非 d1-3B（后者分别为 93.0、76.9）：600M 双向 encoder 血统在文本对表面判别类任务（释义识别、毒性检测）上异常高效，PAWS-X 上对 Decider 2B 领先达 20 分。全表最弱项一致是医学问答（PubMedQA 66.0 上下），高 stakes 领域的决策可靠性尚未建立。评测覆盖也有缺口：Decision Index v0.3 的视觉 split 未公开报告，音频决策基准被作者承认为 open problem——多模态决策能力目前主要靠骨干继承性背书，而非直接测量。

### 600M omni 的战略含义

d1-omni-600M 以 1/4 参数量超过 Decider 2B（78.4 vs 77.1），且只是"首个实验 checkpoint"——这暗示 encoder+adapter 路线做多模态决策的参数效率远高于微调 decoder VLM：判别型任务不需要自回归解码的表达冗余，双向 encoder 的表征在单次前向设定下被更充分利用。音频路径的存在还意味着音频决策能力先于评测标准落地，一旦社区补齐 benchmark，600M 级 omni 决策模型可能率先填补空白。"家族"策略本身也值得注意：3B 给决策质量上限、600M 给 footprint 下限，同一权重来源体系让用户可以在精度与体积间连续换挡。

## 实践启示

1. **端侧选型看状态规模而非单问延迟**：所有设备单问都 <50ms，Orin Nano 也够实时；但 3.4K-token 状态下 Orin Nano 需 1640ms，做长上下文/多帧状态决策必须上 AGX Thor（220ms）或独显级别。
2. **迁移生成式分类管线时重构提示结构**：决策模型下多问题共享状态近乎免费（3 问 ≈ 1.3× 单问），应把逐条调用改为一次状态 + 打包多问，服务端再叠加 packed states 批处理提升吞吐。
3. **自训决策模型优先做数据卫生**：长输入训练、打乱选项顺序、清除 shortcut 样本，这三件事的收益被官方认证大于任何高级训练技术。
4. **不要把 d1 用于高 stakes 判断**：PubMedQA 全表 ~66 分，适合做初筛/分流的第一级过滤，最终判断仍需生成式模型或人工复核。
5. **音频决策需自建评测**：该领域尚无公认基准，接入 d1-omni-600M 的音频路径后必须在自有数据上验证；视觉能力则可参考 LFM2.5-VL 骨干的标准基准。
6. **部署栈可保持供应商中立**：llama.cpp day-one 原生支持覆盖 Apple/AMD/Qualcomm/NVIDIA（含 NVFP4），权重无限制开放，端云两侧均可避免锁定单一硬件生态。

## 关联

- [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2.6B]] — d1-3B 的权重平均来源之一，同厂商生成式侧
- [[entities/liquid-ai-lfm2-5-230m|LFM2.5-230M]] — 同厂商边缘小模型
- [[entities/liquid-ai-lfm2-5-encoders-fast-long-context-cpu|LFM2.5 Encoders]] — d1-omni-600M 的 encoder 骨干家族
- [[concepts/inference-optimization]] — 单次前向决策 vs 自回归生成的推理成本对比维度
- [[concepts/model-distillation-compression]] — 小模型能力上限话题

→ [[raw/articles/liquid-ai-d1-edge-decision-models|原文存档]]
