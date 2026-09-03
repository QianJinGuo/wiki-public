---
title: "GRACE：量化感知训练与知识蒸馏联合优化（ICML 2026）"
created: 2026-08-28
updated: 2026-08-28
type: entity
tags: [icml-2026, quantization, knowledge-distillation, qat, post-training, efficiency, vlm, eth-zurich]
sources: [raw/articles/icml-2026-grace-quantization-aware-distillation]
confidence: 0.72
---

# GRACE：量化感知训练与知识蒸馏联合优化（ICML 2026）

## 核心命题

模型压缩的两个环节——知识蒸馏（变小）与量化（变窄）——**不该分开做，因为它们优化的是同一个表示**。主流流水线"先蒸馏再 PTQ"在 8-bit 尚可、4-bit 明显掉点：蒸馏阶段不知道后面要被量化（学生学到的表示分布尖锐、离群值多，最不适合低比特）；量化阶段已无教师可用（PTQ 的误差无法再被纠正）。GRACE 在**同一目标下联合优化 QAT 与 KD**，让 2B 学生在 4-bit 下逼近 7B 全精度教师，入选 ICML 2026。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]

## 核心洞察：让学生在"戴着镣铐"的状态下学

信息瓶颈（IB）视角：低比特权重限制学生能承载的表示复杂度，学生不应复刻教师全部输出，而应有选择保留影响决策的信息。由此引出三个设计问题：哪些教师输出值得学（教师也会错）、除了输出分布还有什么值得对齐（视觉 token 关系结构）、蒸馏与量化目标怎么配平。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]

## 方法：四个模块

1. **GDKD（置信度门控输出对齐）**：用教师输出分布归一化熵构造门控权重——教师越确定蒸馏权重越高，越犹豫越低。理论由 Fano 不等式支撑（熵高 → 可达到错误率下界高）。蒸馏损失沿用解耦 KD 拆成目标类/非目标类分别加权。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]
2. **RCKA（视觉 token 关系对齐）**：在 LLM 倒数第二层取视觉 token 表示，算 Gram 矩阵，用 CKA 度量师生关系结构一致性（CKA 对正交变换/缩放不敏感，适合学生维度低 + 量化扰动场景）。比对齐数值更稳，注意力可视化显示视觉定位能力更强。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]
3. **自适应 IB 控制器**：把蒸馏/关系/任务三损失配平写成带约束优化（蒸馏损失 ≤ τ 前提下最小化任务损失），拉格朗日对偶上升动态更新乘子 β。是 IB 启发式设计，非严格 IB 实现。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]
4. **分组 LSQ（量化参与训练）**：权重量化以可微形式参与前向，步长 LSQ 学习、对数空间参数化保正、分位数初始化避震荡；分组粒度 128，支持 W8G128 / W4G128。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]

## 实验与结果

- **Qwen2-VL 7B→2B**：4-bit GRACE 学生八项基准均值 68.0，超 BF16 学生基线 4.0 点、超最强 4-bit 方法 SPEED-Q 5.7 点、与 7B 全精度教师差压缩到 3.7 点内。BF16→4-bit 只掉 1.2 点，其他 PTQ 在 4-bit 平均掉在 61 附近。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]
- **Qwen3-VL 8B→2B**：三档精度相对 BF16 基线（67.3）分别提升 9.4/8.6/7.7 点。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]
- **真实 kernel 实测**：INT4 用 AWQ kernel、INT8 用 GPTQ kernel，精度与仿真一致；A100 上 LLaVA-1.5 7B INT4 相比 FP16 在显存/体积/吞吐三项有数倍收益（7B 解码访存受限，位宽下降直接转吞吐）。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]

## 工程价值

无需额外教师微调、不依赖专有校准集；四个模块可单独插入现有 QAT/KD 流程。代码与量化后权重全部开源（github ForeverBlue816/GRACE）。这是**联合压缩**（joint compression）思路的代表——把互为因果的两个压缩步骤放进同一优化目标，而非串行 pipeline。^[raw/articles/icml-2026-grace-quantization-aware-distillation.md]

## 相关

- [[entities/geora-geometry-aware-lora-rlvr-meituan-2026|GeoRA：面向 RLVR 优化几何的 LoRA]]
- [[entities/bonsai-image-4b-quantization|Bonsai-Image-4B 量化]]
- [[entities/anthopic-distillation-behavioural-traits-nature|Anthropic 蒸馏行为特质]]
- [[entities/glm-53-post-training-technical-blog-zhipu-xhs-2026|GLM-5.3 post-training]]

→ [[raw/articles/icml-2026-grace-quantization-aware-distillation|原文存档]]
