---
title: "DLR：视觉语言模型的强化隐空间推理——Decompose, Look, and Reason（EMNLP 2026）"
created: 2026-09-28
updated: 2026-09-28
type: entity
tags: [vlm, multimodal, reasoning, reinforcement-learning, latent-space, emnlp]
sources: [raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26]
confidence: 0.75
provenance_state: extracted
---

# DLR：视觉语言模型的强化隐空间推理——Decompose, Look, and Reason（EMNLP 2026）

> 埃默里大学团队提出 Decompose, Look, and Reason（DLR），缓解多模态 CoT 中「文本推理不断延长，视觉证据逐步弱化」的问题，EMNLP 2026 Main 接收。arXiv: 2604.07518

## 问题：模型是在「看着图推理」还是「看完图靠文本猜」

VLM 在长 CoT 之后，后续推理是否仍由充分、准确的视觉证据支撑，是多模态推理的核心未解问题。已有三条路线各有缺陷：**text-only multimodal CoT** 把视觉压缩为文本描述后在语言空间推理，信息损失不可避免；**interleaved CoT / thinking with images** 在中间步骤引入图像 patch/bbox/crop/zoom/外部工具，增强显式 grounding 但引入工具调用成本且受预定义操作空间限制；**latent visual reasoning** 把中间视觉信息投影到连续嵌入空间，但现有方法依赖局部 ROI/patch 或全程只注入一次视觉 latent——局部 ROI 未必与「当前步骤真正需要的语义证据」一致，且多步推理要求不同阶段检查不同证据，单次注入难以支撑完整推理轨迹。^[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26.md]

## 核心：Decompose–Look–Reason 循环

将复杂视觉推理表示为多步轨迹：第 t 步包含文本 premise p⁽ᵗ⁾、连续视觉 latent z⁽ᵗ⁾、基于该证据的 rationale r⁽ᵗ⁾，最终得答案 a。核心是把「提出需要验证的问题」与「从图像提取对应证据」显式耦合：

- **Decompose**：VLM 根据当前推理上下文动态生成需要验证的 premise/sub-question——回答「接下来需要验证什么」。
- **Look**：visual grounder 以 premise 结束位置的 hidden state 为条件，对图像特征 cross-attention，输出连续视觉 latent——不同于固定 ROI，可编码局部细节、跨区域关系与非局部语义。
- **Reason**：注入 latent 后生成 grounded rationale，继续下一轮分解直到答案——视觉证据成为整个推理轨迹中可动态更新的条件，而非开始时的一次性输入。

概率建模上，文本策略 Pθ 与视觉 latent 策略 Pϕ 联合优化：更准的 decomposition 给 grounder 更明确的条件，更相关的 latent 反过来改善 rationale，二者相互依赖。^[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26.md]

## 三阶段训练：从对齐到连续隐空间强化学习

1. **Visual Grounder Pretraining**：冻结 VLM 主干，训练轻量 grounder；learnable latent queries 条件化提取 + 双向 InfoNCE 对齐 latent 与答案语义。
2. **SFT**：学习结构化 DLR 轨迹（`<premise>`/`<vis_thought>`/`<rationale>`/`<answer>`）， grounder 经后续语言建模损失获得梯度。
3. **Reinforcement Finetuning for Continuous Latents**：将 RL 从离散文本 token 扩展到连续视觉 latent——构造可显式计算策略密度与 importance ratio 的 latent policy，与文本 policy 基于 Dr. GRPO 的 group-relative advantage 联合优化。

关键创新 **Spherical Gaussian Latent Policy（SGLP）**：语义信息多编码在向量方向而非模长（cosine similarity 语义度量），故将视觉表示视为 unit hypersphere 上的语义流形——grounder 预测 L2-normalized 均值方向 μϕ，加高斯扰动后重投影回球面，把探索集中在语义方向上，避免语义变化与模长变化混合。^[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26.md]

## 实验结果（backbone: Qwen3-VL-8B-Thinking）

- 四个视觉中心 benchmark：V* Bench **83.8**（+4.2）、MathVista **82.7**（+3.5）、MMMU-Pro **63.5**（+2.6）、MMStar **76.2**（+3.9），ICoT/LVR/PixelReasoner 等开源方法适配同一 backbone 公平对比。
- MathVista 上相比 LVR 从 77.3 → 82.7，优势归因于多步 premise-conditioned grounding 对反复检查图表/几何关系的任务更契合。
- 消融：最显著组件是 latent policy objective——移除 J_latent 后 MathVista 82.7 → **57.1**，说明仅靠 SFT 的 deterministic 表达不足以发挥隐空间推理，显式连续 latent 探索是必要组件。
- Faithfulness 干预实验：V* 上随机遮挡同等区域准确率 83.8 → 77.7；遮挡 DLR attention 最高区域降至 **36.5**（-47.3 个百分点）——为 grounder 识别区域与预测的功能相关性提供定量证据。
- 案例：weak grounding 导致 overthinking——baseline 在图形规律题上反复试错生成 15,177 tokens 仍答错；DLR 先明确验证目标再取证据，一步到位。^[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26.md]

## 洞察

DLR 的方法论价值有三层：(1) **premise-conditioned latent 提取**把「看什么」交给模型自己决定，摆脱了固定 ROI 与预定义视觉操作空间的限制；(2) **SGLP** 把 GRPO 从离散 token 空间推广到球面连续 latent 空间，是 RLVR 思路在连续表征上的直接延伸，与 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 谱系互补；(3) faithfulness 干预实验（-47.3pp）给出了超越可视化「看起来合理」的因果证据。与 [[entities/laser-acl2026-latent-superposition-visual-reasoning|LASER latent 视觉推理]]、[[entities/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|CoLT latent 思维链]] 同属 latent visual reasoning 方向，DLR 的差异化在多步循环 + RL 优化 latent policy。^[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26.md]

## 关联

- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证强化学习]] — RL 优化谱系
- [[entities/laser-acl2026-latent-superposition-visual-reasoning|LASER latent 视觉推理]] — 同方向方法对照
- [[entities/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|CoLT latent 思维链]] — 同方向方法对照
- [[entities/235b参数也没用港中文等发布7模态数据集专测顶级vlm的感知盲区|VLM 感知盲区评测]] — VLM 感知能力边界

→ [[raw/articles/面向视觉语言模型的强化隐空间推理先分解看再推理emnlp26|原文存档]]
