---
title: "LSTM之父最新97页综述：Agent如何真正学会「自我进化」？"
created: 2026-08-13
updated: 2026-11-20
type: entity
tags: [ai, research, agent, ai-agent, multi-agent, skill, agent-skill, memory, agent-memory, rl, reinforcement-learning, post-training, search, agent-search]
sources: [raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md]
confidence: 0.6
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LSTM之父最新97页综述：Agent如何真正学会「自我进化」？

> WeChat-新智元 | 发布于 2026-08-09 | 评分入库 v×c≥49

## 核心内容

新智元 2026-08-10 07:46 北京 新智元报道 过去两年，self-reflection、self-correction、self-play、agentic RL、skill learning、test-time adaptation、open-ended evolution、self-evolving agent等概念不断涌现。 它们有时指模型继续训练，有时指Prompt或工具自动优化，也有时被直接归入递归式自我改进（Recursive Self-Improvement，RSI）。 问题在于，这些术语描述的变化层级和强度并不相同，却长期缺少一套统一坐标系。 近日，来自吉林大学、KAUST、阿尔伯塔大学和瑞士人工智能实验室等机构的研究团队发布综述，从系统层面重新定义「自我改进」，并将模型训练、记忆演化、工具创建和Agent架构搜索放进同一张地图。配套项目目前已整理312篇相关工作。 论文链接： <https://arxiv.org/abs/2607.13104 项目主页： <https://selfimproving-agent.github.io/ 代码链接： <https://github.com/selfimproving-agent/awesome-Self-Improving-Agents 值得一提的是，这篇综述发布后迅速在海外AI社区获得关注。配套GitHub仓库一周左右已收获200+Stars ，；X上也有不少论文推荐帖也获得了约800个Likes 。这也折射出一个清晰信号，self-improving agent正成为海外研究者和开发者共同关注的新焦。^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md]

## 关键要点

- 原文完整记录：[[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md|原文存档]]
- 关联主题："Agent 架构"、[[concepts/agent-orchestration-patterns]]、[[concepts/harness-engineering-framework]]

## 深度分析

### 「自我改进」的判定边界：改 θ 或 Σ 才算数

综述将基础模型 Agent 形式化为「参数 θ + 脚手架 Σ」：Σ 是围绕模型运行的脚手架，包括 Prompt、Memory、Tools，以及路由、调度和安全约束等控制逻辑^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:44-46]。由此划出的判定边界非常严格——上下文中的临时计划、对话历史和一次性反思只属于运行时状态；只有当系统依据自身执行产生的轨迹、评价或验证结果，持久修改了 θ 或 Σ，并让下一次任务从不同起点出发，才构成真正的 self-improvement^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:48]。

这条边界直接修正了一个流行直觉：self-reflection 与 self-correction 本身不必然等于自我改进。一次任务中「再想一遍」可以抬高当前答案，却未必留下任何跨任务能力；反过来，一条经过验证并写入长期记忆的规则、一个修复后持续复用的工具，哪怕没有更新模型权重，也已经改变了 Agent 未来的行为^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:50]。作者还把该概念放进数十年思想脉络（元学习、Gödel Machine 等），指出基础模型带来的真正变化在于自然语言成为统一的修改介质：错误可以写成批评，经验可以压缩为记忆，工具和工作流可以直接生成与重写，自我修改由此从低层代码与权重搜索，变成更可表达、也更可检查的工程过程^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:52]。

### 两条路线：写进模型，或写进系统

综述把自我改进分为两条主线。第一条是 Foundation Model Improvement：Agent 自己生成训练示例、评价信号，或从真实环境与世界模型中获得探索经验，再通过 SFT、偏好优化或强化学习写回参数——更新较慢、成本较高、不易回滚，但一旦成功，能力可以跨任务共享^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:58-60]。学习信号分三类（自生成示例与轨迹、模型自评信号、环境的真实或模拟经验），Self-Instruct、Constitutional AI、WebRL 分别对应^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:62]。风险同样直接：模型可能把自身错误重新训练进权重，导致偏差放大、模型坍塌或灾难性遗忘，也可能学会钻奖励函数和世界模型的空子，因此参数更新必须配合数据过滤、外部验证、版本管理和回滚^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:64]。

第二条是 Scaffolding Improvement：模型参数保持不变，系统改写 Prompt、整理和淘汰 Memory、选择/修复/创建 Tools，甚至重构整套 Agent 代码，TextGrad、Mem0、Voyager、Darwin Gödel Machine 都属此列（系统级同类机制参见 [[concepts/agent-self-improvement-loops]]）；它更轻量、更透明，更容易验证和撤销，是当前工程实践中最活跃的方向^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:66]。综述梳理的六大应用场景（软件工程、网页导航、游戏、科学发现、具身智能、通用计算机控制）的共同点是 Agent 都能从环境获得反馈，差异在反馈可靠性、获取成本与失败能否撤销；其中软件工程因编译器、单元测试和持续集成成为反馈最密集、最易验证的试验场^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:80-82]。两条路线并非互斥：更合理的系统让脚手架承担快速、局部、可逆的探索，再把反复验证有效的经验蒸馏进参数^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:74]。

### skill：不是第五个组件，而是「可序列化的更新」

综述对 skill 的处理最具架构洞察：skill 不是 Prompt、Memory、Tool 之外的第五个组件，而是一种「可序列化、可复用的更新」——同一个技能可存为工具及调用约定、工作流或记忆，也可固化进权重或控制逻辑；skill 描述的是「保留下来的能力单元」，而不是它被存在哪里^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:68]。由此区分两类技能：对象级 skill 用于完成外部任务；元级 skill 直接作用于 Agent 自身——重写 Prompt、整理 Memory、创建工具或触发参数更新；后者一旦还能改进「如何改进自己」，就触及 RSI 最核心的自指结构^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:70]。

作者同时给热度降温：今天多数所谓 RSI，仍是受目标函数、测试集、沙箱和权限边界约束的「有限自我修改」，距离不受约束的智能爆炸相去甚远；真正该关注的不是科幻式失控，而是让每次更新可测量、可归因、可回滚^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:72]。设计原则凝为八个字：快环探索、慢环固化——反馈嘈杂、任务变化快时优先修改 Prompt、Memory、Tools 和控制逻辑，因为这些变化透明、便宜、可回滚；只有当策略经充分验证、能跨任务迁移时，再蒸馏或微调进模型权重，成为稳定的长期能力^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:100-104]。

### 评测与安全：度量更新轨迹，而不是静态分数

自我改进 Agent 的研究对象是一条更新轨迹，而非静态模型的最终分数。可信实验至少要回答：固定预算下每轮提高多少、能否迁移到未见任务、旧问题是否再次失败、消耗了多少工具调用与人工监督、安全风险是否累积^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:92]。评测需区分可执行指标与 Judge-based 评测，并警惕「同模型既提出改进又负责打分」导致的讨好裁判效应——生成者与评估者应分离，用独立 Judge、隐藏测试和可验证子集校准（参见 [[concepts/agent-evaluation-benchmark-frameworks]]）；机制级评测隔离 Prompt、Memory、Tool 或参数更新的贡献，领域级评测检验分布变化、执行成本和安全约束下的表现^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:94-96]。

安全侧的核心论点是：当 Agent 能够修改 Memory、Tools 乃至自身代码时，安全对象本身也在变化——一次 Prompt Injection 可能不再只是临时干扰，而会被写入记忆或工具逻辑，成为持久漏洞。因此 Critic 应是受治理的基础设施，提出与批准更新不应由同一闭环完全控制，每次修改都要经功能、权限、安全检查；尤其在 Full Scaffolding 与 RSI 场景中，Agent 应被当作受保护环境中的「不可信程序」，所有更新都需经过分层门控^[raw/articles/lstm之父最新97页综述agent如何真正学会自我进化.md:112]。

## 实践启示

1. 先问「留下了什么」再谈「变强了」：任何自称自我改进的机制，都要回答改动是否持久化（写入 θ 或 Σ）、下一次任务是否从不同起点出发。
2. 工程上默认走脚手架路线：改 Prompt/Memory/Tools 便宜、透明、可回滚，是探索的第一现场；只有反复验证有效的策略才值得蒸馏进权重，避免把未经检验的错误烧进参数。
3. 用「更新单元」的视角设计 skill 体系：skill 的价值在可序列化、可复用，同一能力应能在工具、工作流、记忆、权重之间迁移存储；同时区分对象级与元级 skill，为元级更新单独设置权限与审计。
4. 评测从「最终分」改为「轨迹曲线」：记录每轮增益、迁移性、旧任务回归与成本曲线；生成者与评估者必须分离，用隐藏测试和可验证子集防止 Agent 学会讨好裁判。
5. 安全门控要覆盖被更新的对象本身：Prompt Injection 一旦写进 Memory 或工具逻辑就成了持久漏洞，Critic 与审批必须分离，全脚手架/RSI 模式下应把 Agent 当作不可信程序做分层门控。
6. 以「快环探索、慢环固化」作架构评审口诀：健康的自我改进系统会形成「脚手架快速试错 → 验证 → 蒸馏进参数」的协同循环，而不是在改模型与改脚手架之间二选一。

## 相关实体

"Agent 架构" [[concepts/agent-orchestration-patterns]] [[concepts/harness-engineering-framework]] [[concepts/agent-memory-architecture]] [[concepts/agent-memory-system-design]]
