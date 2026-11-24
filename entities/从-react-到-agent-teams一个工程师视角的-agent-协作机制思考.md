---

title: "从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考"
created: 2026-09-01
updated: 2026-11-15
type: entity
tags: [ai, agent, harness, memory, mcp, multimodal]
sources: [raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考


## 从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考

原创 蒋泽林(林曜) 2026-08-31 09:47 浙江

阿里妹导读

两个月的 Agent 开发实践沉淀，从第一性原理出发，探讨 Agent 的本质、当前多 Agent 协作架构的不足，以及如何借鉴人类组织经验设计真正的 Agent Team 机制。（文章内容基于作者个人技术实践与独立思考，旨在分享经验，仅代表个人观点。） ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

一、Agent 的本质：

ReAct 模式即人类智能的工程化抽象

做了两个月的 Agent 开发，我越来越确信：当前 Agent 能真正完成任务，很大程度取决于 ReAct（Reasoning + Acting）模式的出现。ReAct 来自 Yao 等人 2022 年的论文，核心循环是「Thought → Action → Observation」——思考下一步该做什么，执行动作，观察环境返回的信息，再进入下一轮思考。这就是人完成任务时的智能表现：推理、行动、根据结果调整，如此往复。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

这里的 Observation 指 Agent 执行动作后从外部获取的信息——API 返回结果、命令输出、文件内容。它本质上承担了"反馈"功能，让推理始终锚定在事实上，而非模型自己编造。ReAct 相比纯 Chain-of-Thought 的核心优势就在这里：有了外部观察的"接地"，幻觉和错误累积被大幅抑制。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

Agent 本质上就是这个循环的实现，而且可以非常简单。pi-agent[1]（7.6 万 star）的核心 `agent-loop.ts` 约 600 行代码就完成了完整的 Agent 循环。把这个循环构建好就能完成任务，后续的记忆压缩、Skill 加载都是「上下文管理」层面的优化，不改变基本工作原理。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

所以 Agent 可以用任何语言实现。Blade AI 使用 LangGraph 也只是借用了它的状态机编排功能——当需要多个 ReAct Agent 有序串联时，启动一个子 Agent 就是启动一个新的 ReAct 循环。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

二、第一性原理分拆：思考、行动、观察

既然所有 Agent 都是 ReAct 模式加通用大模型，那构建的 Agent 就是通用智能体——理论上可以在任何领域做任何事，区别只在于工具和上下文。沿 ReAct 三环节做分析： ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

**思考（Reasoning）：大模型基座决定，是 Agent 的「智商」 。** 模型的推理能力直接决定 Agent 上限——能不能理解复杂指令、做多步推理、在信息不完整时合理假设。底层模型能力越强，后续可扩展的事就越多，这是不可违背的约束。ReAct 原始实验也印证了这点：同框架换更强模型，所有任务表现直接提升。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

**行动（Acting）：工具集决定，是 Agent 的「手脚」 。** 同一个模型配不同工具就能做不同的事——给自行车只能短距，给汽车就能远行。ReAct 论文在 ALFWorld 中展示：有了动作空间后成功率比纯推理高出 34 个百分点。光有思考没有行动，智能无法体现。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

**观察（Observation）：环境返回的信息，是 Agent 的「感知系统」 。** 必须给大模型真实、结构化的观察信息。如果反馈只说"出错了"但不说为什么，Agent 就像得到模糊评价的人一样陷入混乱——只能瞎猜。工具返回详细的错误描述加修复建议，远比一个错误码有用。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

**工程团队的着力点 ：** 我们无法解决模型的「思考」问题，那是算法团队的事。但可以积极解决「行动」和「观察」——提供更好的工具、返回更结构化的执行结果。这能极大减少幻觉和弯路，不是因为模型变聪明了，而是因为它做决策的依据更充分了。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

三、无状态本质与上下文管理

大模型是无状态的。每次调用都是一次独立的前向计算，模型参数不会因为这次调用发生任何改变。它之所以表现得「像知道上次说过什么」，是因为你把历史又塞回 context 里重新告诉了它——这不是记忆，是每次现讲一遍。给不一样的 context 就得到不一样的输出，仅此而已。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

这里有一个人脑和大模型的根本区别：人今天学会了 Java，晚上突触就长出来了，明天照样会写——**学习这件事物理性地改写了大脑本身** 。大模型完全相反：它跑完一件事、"解决"了一个问题，参数权重一个字节都不会变，下次启动还是那个出厂模型；如果我们不把这次的经验、结论、教训持久化到外部存储、下次通过 context 再喂回去，等于什么都没发生过。所以 Agent 不会"自动变强"——它只在你为它建了外部记忆的那部分变强。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

这也是为什么 Agent 工程绕不开 RAG、记忆压缩、Skill 注入、知识库这一整套东西——它们本质都是在替大模型做那件它自己做不到的事：把经验存起来、需要时精准塞回有限的 context window。学术界的持续学习方向（2026 综述 arXiv 2603.12658）在持续预训练、持续微调（LoRA 变体）、持续对齐上都有进展，但核心难题「灾难性遗忘」——学了新东西旧的就忘——远未根本解决。在这个问题被彻底解决之前，"上下文即记忆"是唯一可靠的工程路径。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

即使有一天持续学习成熟、模型能像人一样在使用中自我进化，「行动」和「观察」仍是工程团队不可替代的战场——模型再聪明，没有好工具就做不了事。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

四、从单 Agent 到 Agent Teams：

必然的进化方向

当前行业聚焦在单 Agent 的构建——完善一个「人」。但未来一定会大规模出现 Agent 间的协作问题，就像人类社会从个体生存到部落、城邦、国家的演化。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

ReAct 的成功来自对「个体智能」的正确抽象。Agent 间的协作同样可以借鉴人类组织经验——几千年试错演化出的管理规则不一定最优，但经过了充分实践验证。2025 年 6 月的 Meta-Team 论文明确引用了组织心理学的「团队反思性」理论来指导 Agent 系统设计，从学术角度验证了这条路。 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]


→ [[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考|原文存档]] ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md]

## 深度分析

### 为什么单循环 ReAct 在团队规模下失效

ReAct 把个体智能抽象成 Thought → Action → Observation 的闭环，Observation 的外部接地抑制了幻觉与错误累积 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:31-33]。但这个抽象只覆盖了「一个 Agent」：它的隐含前提是反馈、记忆、经验都装在同一个 context 里。大模型本身无状态，所谓「记得上次」只是把历史重新塞回 context ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:53-55]，于是当任务复杂到需要多个循环并行时，每个循环都活在各自的 context 里，谁也看不见谁的 Observation——协作这件事在 ReAct 里没有任何对应原语。

文章用两组实验数据把失效量化：手工设计的多 Agent 系统在 9 个测试里有 6 个反而不如单个 Agent ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:55]；去掉协作进化机制后，20 个 Agent 的团队和 1 个 Agent 表现完全一致，多 Agent 的价值 100% 来自协作机制本身，加人不等于加价值 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:57,135]。失效的根因不是模型不够强，而是结构性的：当前 Manager-Workers 架构里 Worker 之间完全隔离，只有指向 Leader 的纵向通道，缺少 peer-to-peer 的横向信息流 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:17,23]——「分工不同」被实现成了「不能说话」。参见 [[concepts/orchestrator-worker-architecture]] 与 [[concepts/multi-agent-collaboration-patterns]]。

### 协作拓扑与角色设计：作者主张什么

拓扑选择上，作者直接引用学术结论：2025 年通信拓扑综述识别了 Flat / Hierarchical / Team / Society / Hybrid 五种架构，越是需要灵活交互、频繁互相求助的场景，全连接 Flat（P2P）结构反而优于层级结构 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:17,33]。他的设计不是在 Leader-Worker 里修修补补，而是重造三个维度：

- **角色**：Leader 必须与 Worker 有真实能力差——更强的模型、更长的 context、更全局的历史访问权——并具备四特征：深度参与方案制定、与 Worker 讨论后再执行、能兜底接手卡住的任务、负责方案变更审查；而不是「同质化的几个 Agent 换个 prompt 就当 Leader」 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:71-73]。
- **流程**：任务启动走显式的「讨论 → 共识 → 执行」三 Phase 协议（Leader Proposal / Worker Feedback / Consensus），把执行层约束前置到规划阶段；人类团队里最贵的沟通开会成本在 Agent 侧几乎不存在，三 Phase 可异步并行、秒级完成 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:89-93]。
- **管理风格**：命令式、带负面暗示的任务描述会让同一个模型产出保守机械、只做最低要求；鼓励式、探索式引导的产出深度和创造性有肉眼可见的提升——因为 RLHF/DPO 训练分布里「命令、否定」语境对应保守免责的回复 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:79-83]。Leader 的语气本身就是工程变量。参见 [[concepts/multi-agent-team-coordination]] 与 [[concepts/agent-role-specialization]]。

### 状态与上下文共享：从孤立 context 到分层资产

文章把「共享什么、放在哪」拆成三层：

- **文件层**：AgentTeams 用 MinIO 提供跨 Worker 的共享文件系统，把大块中间产物放进存储而不是塞进 context，直接降低 token 消耗 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:21]。
- **通信层**：Worker 间横向通信有三种模式——主动广播（push 阶段性成果与共同风险）、被动查询（pull 中间结果与专业判断）、求助升级（先横向找同伴，解决不了再向 Leader 升级） ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:101]。通道由 A2A / ANP / Matrix 提供底座，但只有通道不够，必须叠加通信策略：熟人网络（历史合作过的 Worker 更容易触发通信）、技能索引（按能力域路由请求）、通信预算（每 Worker 每任务上限），否则全连接会随 Worker 数量指数级爆炸 context ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:103]。参见 [[concepts/multi-agent-context-isolation]] 与 [[concepts/context-management-agent-systems]]。
- **知识层**：集体复盘产出三类资产——方法论、协作模式（team playbook）、反模式——并存两处：所有 Worker 可读的团队共享知识库 + 只对当前岗位有意义的角色专属经验 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:131]。这延续了「上下文即记忆」的路线：经验只有持久化并在下次精准塞回有限的 context window，才算真正发生过 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:55-57]。

岗位定义（JD）是另一层共享状态：能力要求、可用 Skill 池、质量标准、汇报关系、SLA 全部显式化，同一个模型配不同 JD 就是不同 Worker；再往上的 Mission 与宗旨，则负责在人类不下场时给自主取舍提供锚点，让能动性不变成失控 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:119-123]。

### 失效模式与恢复

- **方向性返工**：Worker 埋头干到一半才发现方向错。恢复靠前置共识 + 方案变更审查——执行中想改方案必须经 Leader 判断合理性，避免局部最优毁掉全局 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:71,89]。
- **进度黑盒**：Worker 埋头干、Leader 到最后才知道结果。恢复靠跑在 OKR 上的三个自动机制：里程碑触发（KR 应达阶段值时自动检查）、偏离预警（上报进度显示达不到 KR 时自动升级 Leader）、KR 达成即关闭里程碑（避免「完成了再问是否完成」） ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:111]。关键前提是 Worker 必须参与 KR 制定——「KR2 缓存命中率 95% 做不到，因为业务本身长尾访问」这类约束只有执行 Agent 能提供 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:111]。
- **协商缺失**：AgentTeams 案例的缺口正是「通信通道有了，协作语义没有」——没有方案共同讨论、没有 OKR 协商、没有分歧仲裁、没有集体复盘、没有团队演化 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:23-25]。恢复路径是借鉴 Meta-Team 的三层复盘节奏：任务过程中的微复盘（Agent 层互取「你的输出如何影响我决策」的反馈）、阶段交付后的中盘（更新对彼此能力与沟通方式的理解）、任务结束的总复盘（讨论组成是否合理、规则是否修订） ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:133]。
- **隔离导致的重复踩坑**：Worker 互相看不见对方的坑。恢复靠横向广播 + 反模式库——踩过的坑作为 anti-pattern 提前规避 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:101,131]。

值得强调的是，AgentTeams 的工程侧（凭据安全、通信基础设施、共享存储、可观测性、人在环）已解决得相当彻底，瓶颈只在协作机制 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:21-25]——基础设施回答「能不能说话」，还没回答「该说什么、怎么达成共识、说完之后如何沉淀」 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:25]。

## 实践启示

1. **不要用「多开几个 Agent」解决单 Agent 的问题。** 消融数据显示 20 Agent 团队去掉协作机制后与 1 Agent 无差别；先设计协作语义，再扩数量 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:57,135]。
2. **Leader 与 Worker 之间要造能力差，而不是同质化换 prompt。** 更强模型、更长 context、更全局历史访问 + 兜底能力，是 Leader 区别于任务分发器的核心 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:71-73]。
3. **把沟通语气当成工程变量。** 命令式 prompt 激活保守回复分布，鼓励式引导激活探索分布；验收时明确指出做得好的地方，为下次高水平输出建立锚点 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:79-85]。
4. **横向通信要「通道 + 模式 + 预算」三件套一起上。** A2A/Matrix 只解决「能不能说话」，还要设计广播/查询/求助三种模式与通信预算约束，防止全连接把 context 炸掉 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:101-103]。
5. **KR 必须由执行者参与制定。** 单 Agent 递归分解 OKR 是单向灌输；「这个 KR 做不到，因为……」的真实约束只有 Worker 能给，多 Agent 协商式 OKR 才是可执行的目标契约 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:111]。
6. **经验要分层沉淀为团队资产，而不是留在角色内部。** 方法论/协作模式/反模式进共享知识库，角色技巧单独存放；按微复盘→中盘→总复盘的节奏持续触发 ^[raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考.md:131-133]。

## 关联

- 同题异语种孪生页：[[entities/agent-teams-collaboration-mechanism-seven-design]]（归并候选，提案卡 #11 批1）
