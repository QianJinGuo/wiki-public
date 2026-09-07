---
title: "TVIR：面向图文交错报告生成的统一基准与智能体框架 — 南大 × 阿里"
created: 2026-07-24
updated: 2026-09-07
type: entity
tags: [tvir, deep-research-agent, multimodal, text-visual-interleaved, report-generation, benchmark, multi-agent, nju, alibaba]
sources: [raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# TVIR：面向图文交错报告生成的统一基准与智能体框架 — 南大 × 阿里

> 南京大学联合阿里巴巴提出 TVIR（Text–Visual Interleaved Report Generation），一个面向图文交错报告生成的统一基准与智能体框架，首次系统性地评估深度研究智能体的多模态能力。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

## 核心问题

现有深度研究基准与真实分析工作的需求之间存在根本性错位：它们以文本为中心评估，却忽视了真实专业报告中的视觉证据整合。一个能写出流畅文字但生成不准确视觉元素的研究智能体，在高风险决策场景中不可靠。TVIR 重新思考深度研究：它不应被视为纯文本任务，而是一个多模态综合问题，文本和视觉必须被联合生成、联合评估。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

## TVIR-Bench：100 道专家级多模态深度研究任务

TVIR-Bench 是首个专门为端到端多模态研究报告生成设计的综合基准，包含 100 个专家策划的任务（50 中文 + 50 英文），覆盖 10 个主要领域和 3 个复杂度级别。任务设计遵循五大核心原则：角色驱动、需求导向、深度研究、前沿聚焦、多模态整合。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

## TVIR-Agent：四阶段分层多智能体框架

TVIR-Agent 是一个专为图文交错报告生成设计的分层多智能体框架，包含四个核心阶段：^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

### 1. 研究驱动的规划
Planner 解析用户任务，迭代调用搜索和网页抓取工具检索相关信息，综合成结构化大纲。每个大纲单元包含章节标题和摘要、规划的视觉需求、章节级研究笔记（含引用、来源 URL 和关键发现）。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]


### 2. 视觉资源实例化
通过两个专门智能体实现：
- **Image Searcher**：处理肖像、场景、架构图等视觉概念，通过 Google 图片搜索检索候选图像，使用 VQA 工具验证相关性
- **Chart Generator**：处理数据分布或关系的内容，检索数据并验证真实性，生成 Python 绘图代码在沙盒环境中执行

### 3. 上下文感知的顺序写作
Writer 逐章节生成报告，基于当前大纲单元和动态更新的全局上下文（已生成章节的标题、摘要和子章节结构）进行条件生成，同时使用章节级研究笔记作为支撑证据。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]


### 4. 全局索引整理
Polisher 在报告级别处理引用和图片：移除未被引用的参考文献，按 URL 和标准化内容全局去重，重新编号为统一的参考文献列表并更新正文中的引用标记。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]


## 双路径评估框架

TVIR 提出多维度评估框架，包含文本评估（TA）和视觉评估（VA）两个互补组件。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

## 关键实验发现

评估了 9 个深度研究系统（6 个商业系统 + 3 个 TVIR-Agent 变体）：^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

- **TVIR-Agent 整体表现最强**：TVIR-Agent（Claude-4.5-Sonnet）取得最佳整体分数
- **不同变体各有所长**：GLM-4.7 文本评估最高，Claude-4.5-Sonnet 视觉评估最高
- **引用支持差距显著**：TVIR-Agent（GLM-4.7）Citation Support 达 68.64，超最佳商业系统 21.11 分
- **结构性错误更少**：TVIR-Agent 变体产生的结构性错误显著少于商业系统
- **工具使用平衡是关键**：Claude-4.5-Sonnet 采用更平衡的工具使用策略，实现最高图表完成率 94.61%

## 意义

TVIR 为未来可信的多模态深度研究智能体奠定了基础，揭示了当前系统"文本综合远强于视觉整合"的关键局限。^[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代.md]

> ---
> [[raw/articles/南大-阿里提出tvir深度研究agent迈入图文交错时代|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

