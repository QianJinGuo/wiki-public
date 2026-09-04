---

title: "从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai, agent, harness, memory, mcp, multimodal]
sources: [raw/articles/从-react-到-agent-teams一个工程师视角的-agent-协作机制思考]
confidence: 0.75
provenance_state: extracted
---

# 从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考


# 从 ReAct 到 Agent Teams：一个工程师视角的 Agent 协作机制思考

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