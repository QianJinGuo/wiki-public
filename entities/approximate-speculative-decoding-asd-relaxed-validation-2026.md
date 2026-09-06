---
title: "ASD：近似投机解码（Approximate Speculative Decoding）— 预算化最长前缀验证"
created: 2026-09-06
updated: 2026-09-06
type: entity
tags: [inference-optimization, speculative-decoding, llm-engineering, decoding, throughput]
sources: [raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026]
confidence: 0.8
provenance_state: extracted
---

# ASD：近似投机解码（Approximate Speculative Decoding）

> 北航/清华/港大/北大联合团队（arXiv 2608.03447，代码 https://github.com/Kissmetothemoon/ASD）提出。在标准投机解码「首个分歧即截断」规则上引入预算化的近似验证：有选择地接受「大模型本来也几乎想选」的分歧 token，并把其后仍与贪心选择一致的后缀直接复用，免训练、即插即用地提升端到端吞吐。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 问题：标准验证的固有浪费

投机解码（[[concepts/speculative-decoding|Speculative Decoding]]）用轻量草稿模型先猜整块 token，再由大模型一次并行「批改」。标准贪心验证采用二元判断：草稿只要在第一个 token 上与大模型 argmax 不一致，验证立即停止，后面已算好的草稿全部作废——即使大模型已经把整段 logits 都算出来了，这些已付出的计算被白白丢弃。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

关键观察：大模型给后面位置打分时本就假设「前面草稿都成立」。若接受前面那一处小分歧，后面紧跟着的一长串 token 很可能恰好仍是大模型的最优选择——它们本来就已经被算对了。token 级别分歧只是任务质量的「不完美代理信号」，`1776` vs `1,776` vs `\boxed{1776}` 字面不同、答案相同。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 方法：三道闸门控制近似代价

ASD 的核心是「与其在第一个分歧处一刀切，不如在严格可控预算内选择性放行」。当一个草稿 token 与大模型不一致时，先计算其「遗憾值」（regret = 大模型最优选择与草稿选择的概率差），越小说明越无伤大雅。三道闸门：^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

1. **局部遗憾门控（regret gate）**：单个分歧的遗憾值必须足够小，且与「它后面还能挽救多少字」相称。
2. **每块异常次数上限（block cap）**：一个草稿块内最多允许几处分歧，避免单块密集放水。
3. **请求级遗憾预算账本（request-level ledger）**：整段生成中累计接受的偏差总量约束在固定预算内，不让误差随输出长度累积。

三者共同作用使近似被显式量化、可审计。接受一个分歧 token 后，后续草稿字在「包含该分歧的新前缀」下被重新打分，其中一段连续后缀往往仍是大模型的贪心选择——这段后缀直接提交，既不需要额外大模型前向，也不需要新的近似决策，这正是提速的主要来源。预算设为零时严格退化回标准贪心验证。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 实验数字

- Qwen3-14B + DSpark-14B 的 7 个任务：固定负载吞吐平均提升 **7.78%**（区间 3.64%–11.73%），平均每轮接受 token 数从 3.85 → 4.20。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- DSpark / EAGLE3 / Medusa 三种草稿框架共 10 组设置全部正增益（3.05%–15.26%，平均 7.52%），最高提速 15.26%。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 284B 参数 DeepSeek-V4-Flash（8×H20）：验证端接受率提升约 10%–16%。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 验证器新增逻辑每输出 token 仅 0.045–0.083ms；免训练、免微调、免额外大模型前向；新增算术复杂度 O(K)。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 工程形态

ASD 是独立、即插即用的验证器模块：不重写草稿模型、不改变投机解码整体流程，只把标准验证中「首个分歧即截断」替换为「预算化的最长前缀选择」，插入现有流水线即可工作——DSpark、EAGLE3、Medusa 等算法均兼容。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]

## 与既有工作对比

- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark]] / [[entities/lmsys-dflash-speculative-decoding-2026-06|dFlash]] / [[entities/eagle-3-speculative-decoding-optimization|EAGLE-3]] / [[entities/deepseek-dspark-v4-speculative-decoding-deepspec|DeepSpec]] 都在设计更优的草稿模型或采样策略；ASD 不换草稿模型，而是放宽验证规则本身，作为它们之上的可叠加验证器层。^[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026.md]
- 与 [[concepts/inference-optimization|Inference Optimization]] 家族中 Decode 阶段优化的其他思路（量化、投机采样、paged attention）正交。

## 相关实体

- [[concepts/speculative-decoding|Speculative Decoding]]
- [[concepts/inference-optimization|Inference Optimization]]
- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark]]
- [[entities/eagle-3-speculative-decoding-optimization|EAGLE-3]]
- [[entities/lmsys-dflash-speculative-decoding-2026-06|dFlash]]

→ [[raw/articles/approximate-speculative-decoding-asd-relaxed-validation-2026|原文存档]]
