---

title: "分子 LLM 智能体：从架构设计到科学自主"
created: 2026-09-09
updated: 2026-09-09
type: entity
tags: [scientific-ai, llm-agent, molecular, survey, autonomy, workflow]
sources: [raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026]
confidence: 0.75
---

# 分子 LLM 智能体：从架构设计到科学自主

## 为什么分子智能体不同于一般智能体

这篇由香港理工大学、上海人工智能实验室、新加坡国立大学与上海交通大学团队发布的综述（arXiv:2608.23104），系统梳理了分子大模型智能体（Molecular LLM Agents）的发展现状，并从「系统架构」与「科学自主性」两个互补视角，提出面向分子发现智能体的统一分析框架。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

与传统认知不同，分子科学中的智能体远比网页操作或代码生成复杂：它不仅要理解自然语言，还必须准确处理**分子字符串（SMILES/SELFIES）、分子图、二维/三维结构、构象、光谱、模拟结果乃至真实实验数据**，并在整个工作流中持续保证分子身份、立体化学、实验条件和单位的一致性。真实的分子发现并非单次「输入—输出」过程，而是一条不断迭代的科学决策链——从研究目标出发，收集多模态证据、选择分子表示、调用数据库与化学工具、生成筛选候选分子、通过计算模拟或湿实验验证，再根据反馈修改分子与实验方案。然而这种能力也带来新挑战：一个微小的文本错误可能被多步工作流不断放大，最终导致无效计算、昂贵的模拟失败甚至不安全的实验操作。因此，语言流畅、能调用大量工具或多智能体协作，都不等同于真正可靠的[[digbench-scientific-discovery-text-games-agent-benchmark|科学自主能力]]。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

## 四个核心模块：分子智能体应如何构建

论文从系统架构角度，将分子智能体划分为四个相互连接的核心模块：

1. **分子表示与感知**：处理 SMILES、SELFIES、分子图、三维坐标、结构图像、光谱和实验记录等多种表示，并能在不同表示间可靠转换，持续检查原子、键、立体化学、电荷、构象、实验条件与单位是否变化。分子感知不应只是输入编码环节，而应成为连接模型、工具与实验反馈的**状态管理层**。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

2. **以大语言模型为核心的智能体框架**：LLM 作为整个系统的控制器，负责理解科学目标、分解任务、制定计划、调用工具、维护记忆、分析反馈并自我修正。但论文强调：推理、规划、记忆、反思或多智能体协作本身只是技术条件；只有当新观察真正改变后续的分子、假设、计划、工具选择或停止条件时，这些机制才构成有意义的智能体能力。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

3. **分子领域专用工具箱**：连接化学信息学工具、分子数据库、文献检索、对接与动力学模拟、量子化学计算、逆合成规划、光谱分析与自动化实验平台。工具数量不是关键，更重要的是能否正确选择工具、构造合法输入、理解输出不确定性、识别失败、控制成本并记录完整执行轨迹与数据来源。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

4. **学习与优化**：既能优化候选分子，也能从过去任务中学习如何选工具、分配资源、处理错误、调整搜索策略。论文进一步区分「单次任务内的优化」与「跨轨迹、跨研究项目的学习」——后者可能影响未来的研究目标、假设选择与科学议程。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

这种模块化视角也呼应了[[ai-for-science-second-half-ara-amber-jiachen-liu-2026|AI for Science 的工作流范式]]，并强调智能体的价值在于将分散环节连接成完整的科学决策闭环。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

## L1—L4 科学自主等级：以反馈回路衡量自主性

为避免把所有工具调用系统都笼统称为「自主科学智能体」，论文提出一套基于证据的四级科学自主框架，其核心判断标准不是模型大小、工具数量或智能体数量，而是系统能在多大范围内**不依赖强制性人工决策，可靠地闭合科学反馈回路**： ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

- **L1 辅助型/固定工作流智能体**：可检索信息、调工具、提供建议或执行预设定流程，但工具结果不自主改变关键科学决策。
- **L2 自适应计算智能体**：依据数据库、预测模型或模拟工具提供的数字证据，自主修改候选分子、假设、计划、工具选择或停止条件。
- **L3 反馈感知的物理实验智能体**：围绕高层实验目标设计并执行真实实验，用执行状态或测量结果调整后续行动、结果解释或故障恢复。
- **L4 科学议程智能体**：基于跨任务、跨研究项目证据，提出并筛选新的科学问题，自主分配计算与实验资源，持续修订研究方向。

这一划分揭示了「自动化」与「自主性」的区别：机器人按固定程序完成实验可能仍只是 L1；而一个小规模计算工作流若能依据模拟反馈自主修改分子与搜索策略，则可达到 L2。生成若干研究假设也不足以达到 L4，系统必须真正选择、验证并根据结果修改其科学议程。论文将 ChemCrow、ChatDrug、ChemReasoner 等归入 L2 计算反馈系统，将 Coscientist、LLM-RDF、ORGANA 视为具 L3 证据的实验智能体；目前尚无被调研系统充分证明具备跨研究项目制定与修订科学议程的 [[autoresearch-ai-scientific-discovery-l0-l4-challengehub|L4 能力]]。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

## 评测、安全与未来里程碑

在评测方面，论文整理了 MolViBench、MolBench、ChemCost、MDGym、ChemReason-Bench、MADE、ScienceAgentBench 等十二类分子与科学智能体基准，并指出不同基准的「成功率」含义各异（问答准确率、程序执行成功率、结果复现能力或闭环搜索效率），不应被简单放入同一排行榜。评测需同时考察分子与中间状态的有效性、工具选择与调用是否正确、反馈是否真正改变后续决策、失败发现与恢复、多轮优化的持续改进、可复现可追溯性、资源消耗，以及是否出现不安全或越权操作。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

科学自主性越高，安全边界越重要：当智能体从信息检索走向合成规划、仪器控制与实验执行，数字空间错误可能转化为真实物理风险。模型幻觉、长程规划失败、代理指标误导、恶意提示和双重用途问题都可能被自动化实验平台放大。论文提出面向分子智能体的**治理契约**——记录已验证的分子身份、可调用工具、人工审批节点、停止/中止条件及完整审计轨迹。这正对应 [[deeppotential-alibabacloud-agentrun-scientific-ai|智能体运行的科学 AI 评测]] 中对反馈闭合与安全边界的设计关切。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

最后，论文给出五个可检验里程碑：①建立可验证的分子状态契约；②构建可复现、可治理的 L2 计算反馈回路；③在明确安全边界内完成可重复的物理实验；④实现依据测量结果选择后续实验的可靠 L3 迭代研究；⑤通过跨任务、跨研究项目的前瞻性实验为 L4 提供真凭实据。这些里程碑共同指向[[agent-harness-engineering-paradigm|智能体工程基座]]的可持续设计——让每次分子转换都有验证、每次科学决策都有证据、每条执行轨迹都可追溯，使系统自主权始终与其可靠性和安全能力相匹配。 ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]

→ [[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026|原文存档]] ^[raw/articles/molecular-llm-agents-architectural-design-to-scientific-autonomy-survey-2026.md]