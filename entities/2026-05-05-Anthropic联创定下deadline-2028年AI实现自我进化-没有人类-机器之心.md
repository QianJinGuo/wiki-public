---
title: "Anthropic联创定下deadline 2028年AI实现自我进化 没有人类 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心.md|原文存档]]

sha256: 7158d316e486e40c2fe76d198372a0a95333fb6c78de8bfcb84a8262a1fbcf64 ^[raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心.md]

## 摘要

机器之心编译 Anthropic 联合创始人 Jack Clark 在 Import AI 455 期 newsletter 的长文：基于公开基准数据，他判断到 2028 年底出现"无人类参与的 AI 研发"（递归自我改进）的概率超过 60%（2027 年仅 30%），并将此称为一个"不情愿的判断"。核心证据链有四条：一是编码能力——SWE-Bench 从 2023 年底 Claude 2 的 2% 升至 Claude Mythos Preview 的 93.9%；METR time horizon 从 GPT-3.5 的 30 秒（2022）一路升至 GPT-4 的 4 分钟、o1 的 40 分钟、GPT-5.2 High 的 6 小时、Opus 4.6 的约 12 小时。二是 AI 已掌握 AI 研发所需的核心科学技能——CORE-Bench（复现论文）从 GPT-4o 的 21.5% 升至 Opus 4.5 的 95.5%；MLE-Bench（Kaggle 竞赛）从 o1 的 16.9% 升至 Gemini 3 的 64.4%；PostTrainBench（微调开源模型）AI 已达人类基线（51%）的一半左右（25%–28%）^[raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心.md]

三是 LLM 训练优化任务上，Anthropic 报告的平均加速从 Opus 4 的 2.9 倍（2025.5）升至 Opus 4.5 的 16.5 倍、Opus 4.6 的 30 倍、Claude Mythos Preview 的 52 倍（2026.4）——人类研究员做同样任务通常需 4–8 小时才能实现 4 倍加速。四是 AI 已学会管理 AI（Claude Code/OpenCode 中主 agent 监督 sub-agent 的合成团队）。Jack Clark 认为 AI 尚不能提出真正激进的新思想，但 AI 研发大多是"扩大规模→找问题→工程修复"的扎实工程，不需要范式级洞见也能自我推进（佐证：Gemini 系统 Aletheia 自主解决 Erdős-1051 开放问题；但 AlphaGo 第 37 手十年未被超越也是偏悲观信号）。风险方面，他强调递归循环中的误差累积——对齐技术初始精度 99.9%，50 代后降为 95.12%，500 代后降到 60.5%。行业信号：OpenAI 目标 2026 年 9 月前造出自动化 AI 研究实习生，Recursive Superintelligence 刚融资 5 亿美元 ^[raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心.md]

## 关键要点

- Jack Clark 判断依据全部来自公开信息（arXiv、bioRxiv、NBER 论文 + 前沿公司已部署产品），"分形"式向上向右趋势：不同分辨率和尺度上都能观察到 AI 在 AI 研发任务上的进展。
- METR 50% 可靠性时间跨度：GPT-3.5（2022）30 秒 → GPT-4（2023）4 分钟 → o1（2024）40 分钟 → GPT-5.2 High（2025）6 小时 → Opus 4.6（2026）约 12 小时；Ajeya Cotra 认为年底到 100 小时是合理预期。
- AI 优化 CPU 小型 LLM 训练实现：2.9 倍（Opus 4，2025.5）→ 16.5 倍（Opus 4.5）→ 30 倍（Opus 4.6）→ 52 倍（Claude Mythos Preview，2026.4）。
- 反方观点：华盛顿大学 Pedro Domingos 指出 AI 自 1950 年代 LISP 时代就能"构建自身"，真正问题是能否获得递增回报，目前无明显证据。
- 潜在影响：对齐误差累积（99.9% 初始精度 500 代后剩 60.5%）、经济"阿姆达尔定律"瓶颈（如新药临床试验）、资本密集型人力轻型"机器经济"的形成。

## 来源

- 原文: [[raw/articles/2026-05-05-Anthropic联创定下deadline-2028年AI实现自我进化-没有人类-机器之心.md|Anthropic联创定下deadline 2028年AI实现自我进化 没有人类 机器之心]]
