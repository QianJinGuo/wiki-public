---
title: "CDL Solver：用简洁几何描述语言桥接视觉与推理（CVPR 2026，北航）"
created: 2026-08-14
updated: 2026-09-07
type: entity
tags: [geometry-reasoning, multimodal, grpo, rl, mllm, cvpr, beihang, math-reasoning]
sources: [raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CDL Solver：用简洁几何描述语言桥接视觉与推理（CVPR 2026，北航）

## 摘要

CDL Solver 是北京航空航天大学团队（Jingyun Wang, Dian Li, Xiaohan Wang, Gang Liu, Jiahong Yan, Guoliang Kang）在 CVPR 2026 Findings 提出的平面几何问题求解（PGPS）范式。核心洞察：**"解耦"比"大一统"更优雅**——不再让多模态模型端到端联合优化视觉感知与逻辑推理（两头落空），而是用高质、简炼的结构化符号语言 CDL（Conditional Declaration Language）作为中间桥梁，把"读图翻译"与"推理解题"两项任务解耦。仅用 5.5k 数据微调，即在 Formalgeo 测试集达到 85.7% 准确率，超越 Gemini 2.5 Pro（81.8%）、GPT-4o（58.0%），并比此前最佳专用模型 DFE-GPS（23.8 万数据、75.3%）数据量缩减 43 倍。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

## 动机：联合优化的两大困境

传统 PGPS 方法将多模态模型在庞大训练数据上端到端联合微调，存在两个致命痛点：一是视觉感知与逻辑推理的联合优化困境——模型经常看错图，且**过度微调视觉对齐会损害基座 LLM 本身已强大的逻辑推理能力**；二是关键实验发现 LLM 是"隐藏的几何解题高手"——当直接提供完美精确的几何描述符号语言（GT CDL）而不给模型看图时，Qwen3 30B 的解题准确率飙升至 88.4%，远超 Qwen3-VL 30B、Claude-Opus-4.1、Gemini 2.5 Pro 等同等或更大尺寸的多模态模型。结论：LLM 推理能力完全在线，缺的只是一座精确无损而精炼的"桥梁"。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

## 方法与创新

### CDL 三声明语言

CDL（Conditional Declaration Language）包含三种核心声明：**ConsCDL**（图形基本骨架：形状、共线、共圆等）、**ImgCDL**（从几何图提取的物理/数值关系：线段长度、角度、平行垂直）、**TextCDL**（从题目文本提取的物理/数值关系）。关键在"简洁性"——高度结构化的符号语言大幅收窄模型输出搜索空间，使 MLLM Interpreter 训练更高效。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

### 数据重塑：Formalgeo7k-Rec-CoT

原版 Formalgeo7k 数据集存在标注错误、图文不匹配等质量问题；团队派 4 位专家做严格双盲人工校验，重构 Formalgeo7k v2 并引入高质量 CoT 推导步骤，为几何学界贡献高纯度训练基石。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

### 两阶段训练管线

- **Stage 1 — CoT 增强的监督微调**：用 Python 解析器将几何关系自动生成自然语言思维链推理步骤；训练模型按 `<think>推理步骤</think> <cdl>CDL 输出</cdl>` 格式输出。
- **Stage 2 — CDL 匹配奖励机制的 GRPO**：传统 Solution-based Reward（最终答案对=1 否则 0）信号太稀疏（解题正确≠读图全对），小样本下极难收敛。CDL Solver 用贪婪匹配算法逐件对比生成 CDL 与 GT CDL，分别计算 Recall（鼓励完整性）与 Precision（惩罚错误/重复），ConsCDL/ImgCDL/TextCDL 独立计算后综合格式奖励——给 RL 提供高密度、高精度打分机制，极大提升 GRPO 训练效率。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

## 实验结果

- **Formalgeo 测试集**：CDL Solver（Qwen3-VL 8B Interpreter + Qwen3 30B Solver）85.7%，超越 Gemini 2.5 Pro（81.8%）3.9pp、比 GPT-4o（58.0%）高 27.7pp
- **vs 专用模型**：DFE-GPS 用 23.8 万数据端到端训练仅 75.3%；CDL Solver 仅 5.5k 数据（缩减 43 倍）达 85.7%
- **跨域泛化（OOD）**：Unigeo 84.0%、MathVista 80.8%，证明通用性 ^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

## 对多模态技术社区的两个核心洞察

1. **解耦优于大一统**：与其用一个模型同时提升视觉感知与逻辑推理，不如通过高质简炼的结构化符号语言作为中间桥梁，把两项任务解耦，最大限度保留语言模型推理优势。
2. **RL 需要高密度过程奖励**：在数学、编程、几何等强约束领域，二元结果奖励效率极低；设计类似 CDL 匹配的精确过程匹配机制能提供充沛信息流，让 GRPO 在极小数据规模下爆发潜力。^[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang.md]

## 相关实体

- RLHF/DPO/GRPO 对齐 — CDL 匹配奖励是 GRPO 过程奖励的具体案例
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 验证驱动推理]] — 高密度奖励信号设计思路相通
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|AWS GRPO/RLVR 数学推理]] — 同 GRPO 数学推理域
- 推理模型 — 几何推理是推理模型的专项应用

→ [[raw/articles/cvpr-2026-cdl-solver-concise-geometric-description-bridge-beihang|原文存档]]
