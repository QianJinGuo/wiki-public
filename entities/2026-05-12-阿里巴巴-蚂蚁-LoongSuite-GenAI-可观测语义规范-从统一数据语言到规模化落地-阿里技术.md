---
title: "阿里巴巴 蚂蚁 LoongSuite GenAI 可观测语义规范 从统一数据语言到规模化落地 阿里技术"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-12-阿里巴巴-蚂蚁-LoongSuite-GenAI-可观测语义规范-从统一数据语言到规模化落地-阿里技术]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-12-阿里巴巴-蚂蚁-LoongSuite-GenAI-可观测语义规范-从统一数据语言到规模化落地-阿里技术.md|原文存档]]

sha256: c679b3c67bc1b67cd953f41bf5092423e9fa6267f876668b46d1623973e7d919 ^[raw/articles/2026-05-12-阿里巴巴-蚂蚁-LoongSuite-GenAI-可观测语义规范-从统一数据语言到规模化落地-阿里技术.md]

## 摘要

文章介绍阿里巴巴与蚂蚁集团发布的 LoongSuite GenAI 语义规范（SemConv）——在 OpenTelemetry GenAI SemConv 基础上的厂商增强标准。背景是 OTel 的核心价值在于用 SemConv 统一可观测数据语言（如 gen_ai.system、gen_ai.request.model、gen_ai.usage.input_tokens 等标准化字段），但社区标准迭代较慢；2025 年阿里云、阿里控股与蚂蚁集团可观测团队联合启动内部场景语义建模，2026 年在社区 Maintainer 建议下将成果开源至 LoongSuite 品牌，后续择机贡献回 OTel 上游。三项核心增强：一是新增 Entry/Step Span，解决长程 Agent 任务单 Trace 成百上千个 Span 难以观测的问题（Entry Span 还原用户原始输入输出，Step Span 对应每轮 ReAct 的层次化表达）；二是新增 Skill 语义（gen_ai.skill.* 属性），填补 Tool 与 Agent 之间业务功能聚合层的观测空白，解决无法归因功能域、无法统计 Skill 健康指标、多 Skill 并发链路混淆三大痛点，并已向 OTel 社区提交独立 invoke_skill Span 提案；三是 Token 级推理观测——蚂蚁率先构建业界首个覆盖多推理引擎（vLLM、SGLang、TensorRT-LLM）、支持 Token 级深度 Trace 的可观测产品，把请求级观测下沉到每个 Token 的生成耗时、并发干扰与 Top-K 候选分布。规范已在 OpenClaw、QwenPaw、Hermes Agent 等场景落地。^[raw/articles/2026-05-12-阿里巴巴-蚂蚁-LoongSuite-GenAI-可观测语义规范-从统一数据语言到规模化落地-阿里技术.md]

## 关键要点

- OTel SemConv 定位：汇聚全球几十家可观测厂商、数百名专家的采集标准，被社区视为 OTel 的"灵魂"；统一字段让跨业务、跨基础设施、跨观测后端共享同一套分析方法
- Entry/Step Span 设计动机：Agent 长程任务的单 Trace 含成百上千 Span；Top-down 排查路径是先定位 10 轮 ReAct 中哪一轮出错，再深入该轮哪一步
- Skill 语义：gen_ai.skill.* 属性现阶段附着在 execute_tool Span 上（无需新 Span 类型即可落地），集团内另有独立 invoke_skill Span 方案，OTel 社区提案编号 semantic-conventions-genai issue 86
- Token 级观测的推理链：性能异常（单请求慢源于某些 Token 生成慢，大概率是并发干扰）与精度异常（复读/乱码从某个 Token 开始扩散）的本质都在 Token 生成过程，因此定位必须下沉到 Token 级
- Token 级产品观测三类信息：每个 Token 的生成耗时及子阶段、慢 Token 时同实例内多请求并发的相互影响、每个 Token 的 Top-K 候选分布
- 落地范围：覆盖 Agent 层到基建层的全栈可观测，已在 OpenClaw、QwenPaw、Hermes Agent 及电商购物助手等场景落地；同一套 gen_ai.skill 语义可覆盖 OpenClaw、Langchain、Spring AI 等框架

## 来源

- 原文: [[raw/articles/2026-05-12-阿里巴巴-蚂蚁-LoongSuite-GenAI-可观测语义规范-从统一数据语言到规模化落地-阿里技术.md|阿里巴巴 蚂蚁 LoongSuite GenAI 可观测语义规范 从统一数据语言到规模化落地 阿里技术]]
