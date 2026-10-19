---
title: "Scinetics Chain of Science Tokens 与 Chain of Structures：把酶设计变成可验证的结构推理链"
created: 2026-10-19
updated: 2026-10-19
type: entity
tags: [ai4s, science-agent, protein-design, reasoning, enzyme, self-improvement]
sources: [raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026]
confidence: 0.65
provenance_state: extracted
---

# Scinetics Chain of Science Tokens 与 Chain of Structures：把酶设计变成可验证的结构推理链

Scinetics 团队提出「科学 Token 链（Chain of Science Tokens）」：把序列、结构、图像、组学和测量数据表示成模型可读写的科学信息单元，与语言共同构成推理过程——文字提出目标，科学状态承载中间决策，每一步的产物成为下一步的依据。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 一次「思考」对应一次结构变化

该方向的核心主张是让 AI 直接用科学对象思考，而不是先转成文字再推理。以蛋白质结合口袋设计为例：模型读取局部结构、尝试放入一种侧链、检查空间冲突再调整方案；推理不再是纯文本独白，而是一串可被后续计算和实验检验的结构状态。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## Chain of Structures：酶设计的四轮结构链

团队在 Chain of Structures（COS）中选择酶设计作为突破口，把科学 Token 链落实为四轮逐步成形的蛋白质结构：第一轮确定催化核心（按反应需求选择承担关键化学作用的残基、构建局部几何）；第二轮完善活性位点（补充底物识别与支持反应的相互作用）；第三轮构建全局骨架（让蛋白质包围并支撑活性位点，同时保留底物进入空间）；第四轮补全序列与侧链，把局部设计拓展为完整蛋白质。每一轮都产生实际结构状态，后续设计必须尊重已形成的约束。工作由 MIT、哈佛大学、剑桥大学、香港科技大学与 Scinetics 等机构研究人员共同完成，张载熙为通讯作者。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

连接推理与生成是关键：COS 一方面传递明确的残基与位置约束，另一方面利用连续表示把结构训练信号传回推理端，使模型学到的不只是「怎样描述方案」，还有「怎样提出生成器能够实现的方案」。当前四轮流程向前单向推进，更长的推理、回退与纠错仍是待探索方向。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 结果与消融

在酶设计基准对比中 COS 报告六项指标五项排名第一（覆盖功能匹配、结构置信度与结构自洽性）：三级酶分类一致率 EC3 达 57.0%（超过 CodeFP 的 55.3%），pLDDT 79.3，TM-score 0.729。消融显示结构链的价值来自推理与生成的配合：去掉推理链 EC3 降至 53.4%；保留推理链但去掉连续接口则降至 51.8%——只有两者结合才取得最佳表现。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

需要强调的边界：这些结果是计算层面的支持（中间结构可成为设计过程的一部分并影响最终产物），真实酶活性、选择性与稳定性仍待湿实验检验。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## ScienceRSI：从单次推理到跨代学习循环

配套 Perspective 提出 ScienceRSI（科学递归自我改进）：模型与智能体产生候选科学 Token 链，模拟器、数据库、实验与领域专家负责筛选；有证据支持的轨迹进入训练集，失败记录也被保留用于训练更强的下一代模型，新模型再产生新的候选与任务经验，继续推动循环。科学任务提供了具体检验对象——结构是否冲突、模拟是否满足约束、实验能否重复；模型提出候选，外部证据决定哪些经验值得学习，并明确区分预测、模拟与实测。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

远期目标被表述为科学超级智能（Scientific Super Intelligence）：同一个固定模型在专业科学任务上超过垂直模型、在广泛科学推理上超过同规模通用语言模型；Perspective 给出 1000 亿至 3000 亿参数、10 万亿至 30 万亿训练 Token 的探索窗口作为可检验假设。COS 是这一长期方向的早期案例，而非最终模型。^[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026.md]

## 关联

- 概念：[[concepts/ai-self-improvement-bootstrapping|自我改进引导]]、[[concepts/verifier-driven-development|验证驱动开发]]
- 相关实体：[[entities/claude-discovers-art-enzyme-system|Claude 发现 ART 酶系统]]、[[entities/boltzgen-protein-design-amazon-sagemaker-ai|BoltzGen 蛋白质设计]]、[[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026|AI4AI 综述与组合缺口]]、[[entities/jepa-anything-cross-domain-latent-world-model-phai-2026|PhAI JEPA-Anything 跨域潜世界模型]]

→ [[raw/articles/scinetics-chain-of-science-tokens-cos-enzyme-design-2026|原文存档]]
