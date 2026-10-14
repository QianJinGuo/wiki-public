---

title: "DHSA：不动基座权重的推理时偏好对齐（RCM+CAST）"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [ai, alignment, interpretability, inference, llm]
sources: [raw/articles/dhsa-inference-time-preference-alignment-2026-10-12]
confidence: 0.75
provenance_state: extracted
---

# DHSA：不动基座权重的推理时偏好对齐（RCM+CAST）

## 核心思想

DHSA（直接隐状态对齐，Direct Hidden-State Alignment）由成都大学、北卡罗来纳州立大学、复旦大学、澳大利亚凯斯林大学和澳门大学的研究团队提出，将**承载偏好计算的推理时隐状态作为直接适配空间**，实现不动基座权重的推理时偏好对齐 → [[raw/articles/dhsa-inference-time-preference-alignment-2026-10-12|原文存档]]。 ^[raw/articles/dhsa-inference-time-preference-alignment-2026-10-12.md]

问题动机：RLHF/DPO 通过反馈优化模型权重，但后训练提高单次生成成功率的同时，基座模型多次采样能生成更多样的正确答案——一部分对齐问题其实落在「已有行为如何被稳定表达出来」：支持目标的计算已经存在，是什么阻碍它稳定主导生成，又能否直接纠正？ ^[raw/articles/dhsa-inference-time-preference-alignment-2026-10-12.md]

## 方法

- **残差竞争图（Residual Competition Maps, RCM）**：模型每生成一个词，注意力头、前馈模块等组件都向共享残差流写入更新。仅找对生成「影响大」的组件不够——还要知道它在推动目标还是抵消目标。RCM 将外部指定的偏好映射到内部组件间的竞争结构上，量化偏好相关的内部作用。
- **CAST 隐状态控制器**：基于 RCM 的定位结果，在推理时对隐状态施加直接控制（256-16384 个控制参数），使支持目标的计算稳定主导生成。

## 结果

推理时控制即可达到 **DPO 水准**的偏好对齐效果，且完全不更新基座权重——对齐从「训练期权重改写」扩展到「推理期状态调制」。 ^[raw/articles/dhsa-inference-time-preference-alignment-2026-10-12.md]

## 关联

- → [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|RL 对齐算法演进（DPO 家族）]]
- → [[concepts/mechanistic-interpretability|机制可解释性]]（RCM 的残差流归因视角）
