---
title: "Graph Engineering in the Era of LLM Agents：从个体智能到系统智能（综述）"
created: 2026-08-26
updated: 2026-09-30
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

## 深度分析

### 三层智能框架：为什么"模型变强"救不了系统问题
三层智能（模型智能 → 个体智能 → 系统智能）不是能力的三档分级，而是一组严格的上限约束链：模型智能的天花板在于无法跨调用保持状态、无法操作外部世界、无法持续接收环境反馈，于是个体智能用 Agent = Loop（LLM + Harness）把"回答器"变成"能自主追目标的主体"；但个体智能又撞上真实任务的三个硬需求——异构专业分工、并行执行与独立验证、长程持久状态，论文明确指出"单纯增强单个智能体的能力或上下文无法解决这种架构失配"。框架的真正含义在于：每一层的瓶颈都不是上一层的"量变"（更多参数、更长上下文）能解决的，而是需要引入新的组织结构——个体层引入 Harness 与 Loop，系统层引入显式图结构。这也是它区别于"更大模型叙事"的核心立场：智能升级的主战场已经从模型权重转移到系统组织结构。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

### Graph Engineering 三维度：一张图同时回答三个问题
图工程的三个核心部分构成一个完整的调度闭环：任务组织回答"做什么"（目标分解 + 工作流优化，把模糊目标拆成子任务图，隐性思考变成显式结构）、智能体协同回答"谁来做"（能力建模 + 团队组织 + 多智能体通信，其中团队组织是相对稳定的"谁负责什么"，通信图是运行时动态的"此刻谁该跟谁说话"——一静一动两层结构值得注意）、运行时状态管理回答"做得如何"（状态记录 + 故障定位 + 失败恢复）。第三个维度被论文称为"整个系统智能中最关键、也最容易被忽视的部分"：前两个维度决定系统跑得好不好，第三个决定系统错了之后还能不能活。跨领域扫描显示成熟度不均——任务组织与团队工程已常见，显式运行时状态管理渐增，而持久系统演化（永久性修订组织本身，而非仅在预定结构内适应执行）仍然罕见，这正是当前多智能体系统与真正"自演化组织"之间的差距所在。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

### 从 Loop 到 Graph 的工程演进逻辑
Loop Engineering 与 Graph Engineering 的关系不是替代而是尺度的跃迁：Loop 在个体尺度组织"计划-行动-观察-验证-调整"的持续闭环，Graph 在系统尺度组织任务、智能体与状态之间的关系网络。演进逻辑的关键在于，当任务复杂度超过单个 Loop 的组织能力时，问题不再是"把 Loop 写得更好"，而是"把多个 Loop 的关系显式化"——这正是"用图来表达关系是天然结构"的由来。这一演进与库内 [[entities/graph-engineering-loop-to-graph-tencent|腾讯从单循环到多节点编排]] 的叙事一致：先有经过验证的单 Loop 能力，再把 Loop 节点化、编排化。理解这一点可以避免常见的过早架构化错误：在 Loop 还没跑稳之前就上多智能体图，只会把单体的不确定性放大成系统的不确定性。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

### 为什么 Ontology Engineering 是未来方向而非可选项
图工程解决了"连接关系"，却保证不了"语义一致"——同一概念不同表示、目标/证据/状态定义不一致、沟通歧义，这些语义障碍在异构智能体协作时会随规模恶化。论文的解法是用共享、机器可解释的本体（类/实体、属性/关系、约束/公理，以 RDF/RDFS/OWL 编写）统一各方对核心概念的定义，且强调本体"不仅是文档，更是可执行的逻辑模型"——这使它区别于 conventional 的架构图或团队章程。把它列为未来方向的理由在于：图工程构建系统骨架，本体工程提供语义地基，骨架可以先搭（图结构可以随任务动态演化），地基却必须先于大规模协作存在（否则每个智能体都在用自己的私有语义解释协作消息）。这与 [[entities/palantir-foundry-closed-loop-ontology-open-source-mvp-2026|Palantir Foundry 的闭环 Ontology 三层]] 与 [[entities/enterprise-ai-ontology-agent-knowledge-governance|企业 AI 本体驱动 Agent 的治理实践]] 形成呼应：学界方向与工业界已落地的范式指向同一点。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]

## 实践启示

1. **用"三个硬需求"作为升级触发器**：当任务天然需要异构专业分工、并行执行 + 独立验证、或跨长程的持久状态三者之一时，单 Loop 已到架构上限——此时应从 Loop Engineering 升级到 Graph Engineering，而不是继续堆上下文或参数。三者都不满足时，留在单 Loop。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]
2. **先做任务组织，再做智能体协同**：先拆出子任务图（什么能并行、什么必须等、怎么验证），再据此分配智能体角色。顺序颠倒（先定团队再定任务）会导致通信结构与实际依赖脱节。
3. **优先补运行时状态管理，而非加智能体数量**：状态记录、故障定位、失败恢复是系统智能中最关键也最易被忽视的部分。一个 3 个智能体 + 完整状态管理的系统，通常胜过 10 个智能体 + 无状态管理的系统——复杂长程任务中出错是必然的，容错能力决定成败。^[raw/articles/graph-engineering-survey-system-intelligence-paper-2026.md]
4. **团队组织与通信结构分层设计**：用相对稳定的"谁负责什么"承载职责边界，用运行时动态的通信图承载协作路由，不要把两者混在同一个静态配置里。
5. **尽早引入轻量本体治理**：在多智能体系统扩到跨团队/跨工具之前，统一目标、证据、状态的定义（哪怕只是共享的 schema 文档），避免语义障碍随规模恶化；Ontology 的本质是"可执行的逻辑模型"，不是又一份架构文档。参见 [[entities/enterprise-ai-ontology-agent-knowledge-governance|企业 AI 本体驱动 Agent]]。
6. **把"持久系统演化"作为明确的演进目标**：当前多数系统只在预定组织结构内适应执行；若系统需要长期运行，应设计机制让组织结构本身（任务图、团队、通信拓扑）能被运行证据永久修订，而不只是临时重排。

## 相关
Graph Engineering 主题已在库内多实体覆盖（如 [[entities/graph-engineering-loop-to-graph-tencent|Graph Engineering：从单循环到多节点编排]]、[[entities/graph-engineering-codez-14-step-zhixin-2026|Codez Graph Engineering 精读]]），但本篇为系统级综述论文原文，提出三层智能框架 + Graph Engineering 三维度 + Ontology Engineering 未来方向的完整体系，属于综述母版。Ontology Engineering 方向与 [[entities/palantir-foundry-closed-loop-ontology-open-source-mvp-2026|Palantir Foundry 闭环操作范式（Ontology 三层）]]、[[entities/enterprise-ai-ontology-agent-knowledge-governance|企业 AI 本体驱动 Agent]] 呼应。→ [[raw/articles/graph-engineering-survey-system-intelligence-paper-2026|原文存档（论文 PDF）]]
