---
title: "表征平等（Representational Equality）：9 个大模型 59 国价值观社会模拟评测（CL 2026）"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [llm-evaluation, evaluation-methodology, representational-equality, cross-country, value-simulation, synthetic-respondents, fairness, alignment, world-values-survey]
sources: [raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026]
confidence: 0.85
provenance_state: extracted
---

# 表征平等（Representational Equality）：9 个大模型 59 国价值观社会模拟评测（CL 2026）

清华大学联合复旦大学、南开大学与上海交通大学基于世界价值观调查（World Values Survey，WVS）对 9 个开源大模型开展系统评测，覆盖 59 个国家、160 道价值观问题、11 个价值维度和 2,420 个可比较子群体单元，提出并量化「表征平等（representational equality）」：把评估重点从「模型平均有多准」扩展到「这种模拟能力如何分布在不同国家之间」。论文 2026 年 7 月被 Computational Linguistics（CL）接收，题为《Representational Equality in Cross-country Value Simulation: A Systematic Analysis of Large Language Models》。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 评测框架：三个指标 + 一份综合分

给模型指定国家、年龄、性别等人口特征，让其以相应身份回答社会调查问卷，由此生成的「合成受访者（synthetic respondents）」已成为大模型社会模拟的重要路径，被用于预调查、实验设计测试与群体态度模拟。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

三件套指标设计如下：① 模拟准确性用 Jensen–Shannon 散度（JSD）衡量模型预测回答分布与真实调查分布的差异，以 1 减 JSD 作为准确性；先在每个价值维度内部计算平均准确性，再对 11 个价值维度赋相同权重汇总，避免题目数量多的维度主导结果。② 表征平等用各国准确性的变异系数（CV）作为主指标，CV 越低说明各国准确性离散程度越小、跨国表征平等越高，并用极差、最值比和基尼系数做稳健性检验。③ Accuracy–Equality（AE）综合指标把平均模拟误差与表征平等做几何平均（越低越好），因为「对所有国家都差但差距小」的模型会拿到虚高的平等分。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 核心发现：准确性与平等性排序不一致

平均模拟准确性最高的是 GLM-4-9B，但其跨国表征平等相对较低；ChatGLM3-6B 平均准确性并非最高，却具有 9 个模型中最高的表征平等水平，并在 AE 综合指标上取得最佳结果。部分模型系列中参数规模增加能提高平均准确性，却没有稳定改善表征平等——总体指标领先不能推导出模型对不同国家具有同样稳定的模拟能力。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

跨国差异呈明显区域格局：澳大利亚、北美和欧洲国家总体更容易被准确模拟，埃及、约旦、伊拉克等中东国家持续表现出更大模拟偏差。多数模型中人均 GDP、互联网使用率、全球创新指数（GII）与模拟准确性正相关；世界银行六项治理指标也普遍正相关；文化维度上，权力距离较低、个人主义程度较高、放纵程度较高的国家被模拟得更准确。已有研究也发现模型更容易反映 WEIRD（西方的、受过教育的、工业化的、富裕的、民主的）社会观点。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 非对称趋同：弱表征群体向强表征群体靠拢

美国 vs 墨西哥的对照最有说服力：题项询问「对失业或找不到工作有多担忧」时，真实调查中墨西哥受访者主要集中于「非常担忧」，美国选择该选项的比例明显较低；但 Llama-3-8B 对美墨人群给出的预测分布彼此接近，且都更接近美国的真实价值分布。扩展到全部 160 道题后，约三分之二的题目中墨西哥人群的模型预测更接近美国的真实分布，且当两国真实分布差异较大时趋势依然存在。这与「群体异质性压缩」相呼应，并进一步指向**非对称趋同**：表征不平等不仅表现为国家间准确性差异，也表现为模型捕捉弱表征群体独特价值模式的能力不足，使预测向其表征更充分的价值模式收敛。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 四类干预策略：准确性与平等往往不同步

研究在统一评测框架内比较了上下文适应（母语提示、相关群体信息补充）与模型训练（持续后训练、偏好对齐）两条路径，每项实验同时考察平均准确性、表征平等与 AE 指标。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

| 干预 | 对平均准确性 | 对表征平等 |
|---|---|---|
| 母语提示（中/阿/西替代英语提问） | 多数语言—模型组合提高，增益在模型与语言群体间差异明显 | 因模型而异；部分模型收益集中在特定语言群体、原本准确群体被替换，不平等未实质缩小 |
| 补充相关群体信息（在少量人口属性画像外补充群体背景与价值信息） | 多数设置改善 | 多数设置同步改善；直接检索效果更明显，模型自改写查询的扩展检索可能重新引入已有先验、削弱改善 |
| 持续后训练（阿拉伯语/日语/俄语/塞尔维亚语/泰语/土耳其语） | 提高相应语言组层面平均准确性（如阿拉伯语国家相对准确性变化约 −1.2% ~ 3.4%），但国家与题项层面分布不均 | 不保证同步提高 |
| 偏好对齐（DPO / GRPO，人工标注 vs GPT 标注偏好数据） | 相对基础模型变化缺乏一致方向；人工标注偏好数据通常更好保持准确性 | 未表现出系统性改善 |

关键结论：一项干预即使提高总体模拟准确性，收益也可能集中于特定国家或语言群体，从而未改善甚至可能降低跨国表征平等。因此无论是比较模型的社会模拟能力，还是评估新的提示、训练或对齐策略，都需要把准确性与表征平等共同纳入评价，并识别性能提升发生在哪些国家与群体。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 方法论意义

这篇文章的价值不在「某个模型更准」的排名，而在把**分布层面的评估公平性**做成可计算指标：用 CV/极差/基尼/最值比多指标交叉验证平等性，再用几何平均构造 AE 以避免「总体差但差距小」的伪平等，并用四类干预的对照实验证明「准确性提升 ≠ 平等改善」。这套思路与 [[concepts/evaluation-harness-design|评测 Harness 设计]]、[[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 关心的「评测指标如何被优化目标反噬」同源，可直接迁移到 [[entities/ai-evals-methodology|AI Evals 方法论]] 中的分组公平性检查，也与 [[entities/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026|PolyWorkBench 跨语言 Agent 基准]] 形成「跨语言/跨群体评测」的同一主题簇。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

落到工程实践上，最可操作的结论是：**分组指标必须与总体指标并列报告**，且要注意检索增强式补偿的副作用——自改写查询会把模型先验带回来，而外部信息对「数字痕迹稀缺群体」本身就不可得，这会让补偿策略在最需要它的群体上失效。^[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026.md]

## 相关条目

- [[entities/multilingual-ai|Multilingual AI]] — 语言覆盖与跨语言能力的不均衡是表征不平等的直接来源之一
- [[entities/scoped-hiring-process-aware-fairness-multi-agent-emnlp-2026|流程感知公平性（EMNLP 2026）]] — 另一个「总体指标掩盖分组差异」的实证案例
- [[concepts/ai-ethics-responsible-ai|AI 伦理与负责任 AI]] — 价值观对齐与表征公平的上位框架

→ [[raw/articles/llm-representational-equality-cross-country-value-simulation-cl-2026|原文存档]]
