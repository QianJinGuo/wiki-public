---
title: "Graph Engineering in the Era of LLM Agents：从个体智能到系统智能（综述）"
created: 2026-08-26
updated: 2026-09-07
type: entity
tags: [graph-engineering, ontology-engineering, system-intelligence, survey, multi-agent, harness, loop-engineering, arxiv]
sources: [raw/articles/graph-engineering-survey-system-intelligence-paper-2026]
confidence: 0.88
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Graph Engineering in the Era of LLM Agents：从个体智能到系统智能（综述）

> 15 家机构联合综述（吉林大学主导，63 页，arXiv:2608.21156）一手论文原文：提出三层智能（模型/个体/系统）演进框架，引入 Graph Engineering（任务组织/智能体协同/运行时状态管理）作为从个体智能到系统智能的桥梁，并以 Ontology Engineering 作为下一代系统智能的未来方向。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## 核心命题：三层智能演进
LLM 已从语言生成模型演化为能解决复杂长程任务的自主智能体，伴随一系列工程范式：Prompt Engineering（激发能力）、Context Engineering（管理信息访问）、Harness Engineering（组织外部工具资源）、Loop Engineering（持续反思与自我改进）。但真实世界任务复杂度上升后，个体智能的根本局限浮现——许多任务天然需要异构专业知识、相互依赖的子任务、并行执行、独立验证和持久状态，超出任何单个智能体的组织能力。智能必须分布到多个专职智能体并在系统层面组织，即**系统智能（System Intelligence）**。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

**三层智能框架**：模型智能（Model Intelligence，基础模型+Prompt/Context Engineering，天花板=无法跨调用保持状态/操作外部世界/持续接收环境反馈）→ 个体智能（Individual Intelligence，Agent = Loop（LLM+Harness），把模型变成能自主追目标的个体智能体，如 Claude Code/Codex）→ 系统智能（System Intelligence，把智能分散到多个专职智能体在系统层面组织）。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## Graph Engineering：从个体智能到系统智能的桥梁
不同于此前主要优化个体交互或智能体级行为的范式，Graph Engineering 聚焦于构建显式、动态、可演化的图结构来组织任务、智能体与运行状态。三个核心部分：

- **任务组织（做什么）**：目标分解（Goal Decomposition）+ 工作流优化（Workflow Optimization），把模糊目标拆成可调度/可执行/可修改的子任务图，让什么能并行、什么必须等、怎么验证都一目了然。
- **智能体协同（谁来做）**：智能体能力建模 + 智能体团队组织（相对稳定的"谁负责什么"）+ 多智能体通信（运行时动态的"此刻谁该跟谁说话"）。
- **运行时状态管理（做得如何）**：状态记录 + 故障定位 + 失败恢复——整个系统智能中最关键也最易被忽视的部分，是系统的记忆和容错机制。

**系统演化（System Evolution）**：跨应用领域成熟度不均——工作组织与智能体团队工程已常见，显式运行时状态管理渐增，但持久系统演化仍罕见（多数系统只在预定组织结构内适应执行，而非永久修订组织本身）。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## 未来方向：Ontology Engineering（本体工程）
图工程让关系显式，却保证不了系统中各方对同一概念理解一致——缺少统一语义，多智能体协作面临严重语义障碍（同一概念不同表示、目标/证据/状态定义不一致、沟通歧义）。本体工程用共享、机器可解释的实体/关系/约束模型统一目标、能力、证据与状态的定义。一个本体精确定义领域内三件事：类/实体、属性/关系、约束/公理，用 RDF、RDFS、OWL 编写——让本体不仅是文档，更是可执行的逻辑模型。它确保所有智能体对目标、证据、任务完成等核心概念有完全一致的理解。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

**图工程 vs 本体工程**：图工程管"连接关系"（谁连谁），本体工程管"概念一致性"（连接在语义上意味着什么、如何被一致解读）。图让关系显式，本体让含义一致，两者互补——图工程构建系统骨架，本体工程提供语义地基。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

其他关键方向：目标形成与价值对齐（Goal Formation and Value Alignment，让系统在共享目标上对齐）、共享语义与世界锚定（Shared Semantics and World Grounding）、衡量系统智能（Measuring System Intelligence，评估对象从单模型/单智能体扩展到多智能体系统的协调质量/任务完成度/状态一致性/演化能力）。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## 评估、工程生态与应用
评估按三层智能组织（模型/个体/系统各自基准与评测原则）。应用覆盖七类领域：软件工程与 IT 运维、科学发现与实验室自动化、医疗健康与临床决策支持、企业工作流与数字组织、通用数字智能体与个人自动化、社会与经济模拟、跨领域发现。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## 相关
Graph Engineering 主题已在库内多实体覆盖（如 [[entities/graph-engineering-loop-to-graph-tencent|Graph Engineering：从单循环到多节点编排]]、[[entities/graph-engineering-codez-14-step-zhixin-2026|Codez Graph Engineering 精读]]），但本篇为系统级综述论文原文，提出三层智能框架 + Graph Engineering 三维度 + Ontology Engineering 未来方向的完整体系，属于综述母版。Ontology Engineering 方向与 [[entities/palantir-foundry-closed-loop-ontology-open-source-mvp-2026|Palantir Foundry 闭环操作范式（Ontology 三层）]]、[[entities/enterprise-ai-ontology-agent-knowledge-governance|企业 AI 本体驱动 Agent]] 呼应。→ [[raw/articles/graph-engineering-survey-system-intelligence-paper-2026|原文存档（论文 PDF）]]
