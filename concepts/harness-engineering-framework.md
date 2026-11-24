---
title: Harness Engineering 框架
created: 2026-04-27
updated: 2026-11-09
type: concept
tags: [concept, agent, architecture, harness, prompt-engineering, context-engineering]
related:
  - [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]]
  - [[entities/openclaw-prompt-context-harness|OpenClaw Prompt/Context/Harness]]
  - [[entities/claude-code-prompt-context-harness|Claude Code Prompt/Context/Harness]]
  - [[entities/alibaba-skill-up-agent-skill-evaluation|Alibaba Skill-Up CLI]]
sources:
  - raw/articles/harness-engineering-framework
  - raw/articles/peking-university-harness-engineering-taming-agent-2026
  - raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09
confidence: high
---
# Harness Engineering 框架
## 概述
AI 工程领域三次重心的演进：
| 阶段 | 核心问题 | 关键词 |
|------|------|------|
| Prompt Engineering | 怎么说 | 角色/风格/Few-shot/分步/格式约束 |
| Context Engineering | 给什么 | RAG / Skills渐进式披露 / 信息选择与裁剪 |
| Harness Engineering | 别跑偏 | 持续监督/纠偏/验收/约束/失败恢复 |
三层包含关系：**Prompt ⊂ Context ⊂ Harness**。
## 核心方程
> Agent = Model + Harness
> Harness = Agent - Model
**Harness = 除模型外的所有东西**。它决定了模型看到什么、能做什么、按什么规则做、做错了怎么纠偏，以及最后如何把能力稳定地交付出来。
## 六层结构
### 1. 上下文管理
角色与目标定义 / 信息选择与裁剪 / 上下文结构化组织（分层）
### 2. 工具系统
给模型什么工具 / 什么时候调用工具 / 工具结果如何重新喂回模型
### 3. 执行编排
理解目标 → 判断信息是否足够 → 必要时获取外部信息 → 基于结果继续分析 → 生成输出 → 检查输出是否满足要求 → 不满足则修正或重试
### 4. 状态与记忆
当前任务进行到哪一步 / 哪些中间结果应该保留 / 哪些内容应该形成长期记忆
### 5. 评估与观测
输出验收 / 环境验证（实际操作页面/跑交互） / 自动测试 / 过程观测 / 质量归因
### 6. 约束、校验与失败恢复
约束 / 校验 / 恢复（分析原因→重试/切换路径/回退）
## 关键实践
### Anthropic: Context Reset vs Compaction
- **Compaction**：同一Agent，历史变短，心理状态延续
- **Reset**：直接换干净上下文的新Agent，交班时交接清楚
- 对于某些模型（如 Claude Sonnet 4.5），Reset 才能真正"清空包袱、重新出发"
### Anthropic: Generator + Evaluator 分离
- 模型自评偏乐观 → 生产与验收必须分离
- Evaluator 不是读代码打分，而是实际操作页面/跑交互
### OpenAI: 渐进式披露
- AGENTS.md 只有 ~100 行（目录页），详细文档在仓库里
- CI 自动校验文档新鲜度和交叉引用
- "文档园丁" Agent 定期扫描过时文档
### LangChain: Harness 改造效果
- 底层模型完全不变，仅改造 Harness
- Terminal Bench 2.0：52.8 → 66.5（Top30 → Top5）
## 核心洞察
> 真正决定 AI 产品上限的，也许是模型；
> 但真正决定 AI 产品能否落地、能否稳定交付的，往往是 Harness。
> AI 落地的核心挑战，正在从"让模型显得聪明"，转向"让模型在真实世界里稳定工作"。
## Harness Engineering 与上下文分层：信息治理的深层机制

Harness Engineering 之所以能解决"别跑偏"的问题，根本上是因为它建立了一套**信息分层治理机制**——不同类型的信息在不同的生命周期阶段、以不同的策略被处理，而不是一股脑塞进上下文窗口。

上下文分层的必要性源于一个根本矛盾：**Context Rot（上下文腐化）**。当无关内容占据上下文的多数篇幅时，Agent 的决策质量会系统性下降，无论模型有多强大。这不是能力问题，而是信号噪音比问题——关键信息被稀释了。^[raw/articles/harness-engineering-framework.md]

上下文分层体系通常包含五个层次：

**常驻层（Permanent）**：身份定义、核心约束、项目级约定。这层信息的特点是使用频率极高、稳定性极强，每次对话都需要存在，但内容短小精悍。常驻层如果太重，Context Rot 会在第一轮就发生。

**按需加载层（On-demand）**：Skills 和领域知识文档。默认只注入名称和描述（OpenClaw 的渐进式披露实践），任务真正需要时才读取完整内容。这个设计避免了上下文膨胀，同时保证了能力按需可得。

**运行时注入层（Runtime）**：当前任务的目标描述、用户最新反馈、工具调用结果。这层信息只存在于当前任务的生命周期内，任务结束即释放。

**记忆层（Memory）**：跨会话积累的经验、被否决的方案、已确认的结论。当前任务需要参考历史判断时从中检索，用完即回到存储状态而非持续占用上下文。

**系统层（System）**：确定性逻辑、编译器/类型检查器输出、lint 结果。这层不由模型生成，而是由确定性系统产生，是验证模型输出的外部基准。

上下文分层与 Harness 六层结构之间的关系在于：上下文分层是第 1 层（上下文管理）的具体实现方式，而 Harness 的其他五层（工具系统、执行编排、状态与记忆、评估与观测、约束校验与失败恢复）则在这一分层体系之上构建了完整的运行保障。Thin Harness Fat Skills 哲学在这一语境下的含义是：框架层（上下文分层机制）应该薄，而能力层（Skills/知识）的丰富度决定系统的实际智能水平。

## Generator-Evaluator 模式：反馈环的本质与自我评估陷阱

Harness Engineering 中最被低估的设计模式是 **Generator-Evaluator 分离**（Anthropic 实践），它揭示了 AI 系统反馈环设计的核心原理。

**为什么 Generator 和 Evaluator 必须分离？** 模型在生成内容时是 Generator，在评估自己生成的内容时存在系统性偏差——自我评估偏乐观。模型会为自己的输出辩护、解释合理性、放大成功面、淡化失败面。这是 LLM 的固有倾向，而非个别模型的问题。^[raw/articles/harness-engineering-framework.md]

Generator-Evaluator 分离的核心要点：
- **Generator** 负责推进任务：理解目标、调用工具、生成中间结果和最终输出
- **Evaluator** 负责验收结果：不读 Generator 的"心理活动"，直接操作环境验证实际效果
- **反馈环**：Evaluator 的验证结果重新注入 Generator 的下一轮决策

这与软件工程中"写代码的人不该自己测试自己的代码"原则完全对应。单元测试由独立的测试工程师编写，才能发现开发者因"确认偏见"而忽略的边界 case；同样，Agent 的输出由独立的 Evaluator 验证，才能避免 Generator 的自我辩护效应。

**Evaluator 的三种实现形态**：

第一种是**环境验证型**（Anthropic 实践）：Evaluator 实际执行操作而非读取文本。Generator 生成修改文件的代码，Evaluator 用 Playwright 或类似工具实际访问修改后的页面，确认功能正常运行^[raw/articles/harness-engineering-framework.md]。这种方式最接近真实用户场景，但实现成本最高。

第二种是**确定性工具型**：Evaluator 调用编译器、类型检查器、lint 工具、测试套件等确定性系统，机械地验证输出是否符合规范。Martin Fowler 强调的"架构规则交给 linter 和 CI 机械执行"正是这一形态。这种方式的优势在于零主观性，但覆盖范围受限于已有工具的完备性。

第三种是**独立模型型**：用另一个模型（通常同型号但不同实例，或更弱的型号）专门做验证评估。OpenAI 在某些任务中使用了这一模式。优势是实现相对简单，劣势是引入了额外的模型调用成本和两个模型之间的一致性问题。

**反馈环的稳定性条件**：Generator-Evaluator 之间的反馈环要稳定工作，需要满足三个条件：

第一，**验证结果必须可操作**。Evaluator 给出"失败"的判断后，Generator 必须有明确的重试路径或回退方案。模糊的"效果不好"不如具体的"类型检查报错 line 42"有价值。

第二，**环内信息不能腐化**。Generator 和 Evaluator 共享的上下文信息（如任务目标、历史决策）如果出现漂移，反馈环会逐步偏离正确方向。这要求 Harness 提供稳定的状态管理，而非依赖 Generator 的自我记录。

第三，**最大循环次数必须有上限**。反馈不是无限的——如果 Generator-Evaluator 循环 N 次仍未收敛，系统应该主动终止并上报，而不是无限重试。Harness 的第 6 层（约束、校验与失败恢复）提供的就是这一层的保障。

Generator-Evaluator 模式与 Harness 六层结构的对应关系：Generator 主要工作在第 2 层（工具系统）和第 3 层（执行编排），Evaluator 主要工作在第 5 层（评估与观测），两者通过第 4 层（状态与记忆）共享信息，并通过第 6 层（约束、校验与失败恢复）获得兜底保障。

## 生产实践补充：货拉拉风控分析 Agent 的四支柱（2026-09）

上面六层是框架层；货拉拉技术这篇给的是**同一套结构在一个具体生产场景里的落地清单**——团队用团队 Agent 做 text2SQL 与提示词工程执行分析（把写 SQL、查数据、归因分析的重复事务从人身上剥走），随后撞上上下文爆炸、任务超时、幻觉、权限管理，这些问题继续优化提示词解决不了，才开始做 Harness。规模背景：网关日志原始数据一天**数 T 级**，清洗后仍近 T 级；分析任务接入覆盖 **1k+ API**。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**先回答「你需要 Harness 吗」——三个明确不需要的情景**：①只要短期、临时、少量单轮问答或资料检索，模型 + System Prompt + RAG 就够，上 Harness 反而增加复杂度；②预算充足、直接上最强模型，Harness 可能只会降低上限；③架构从一开始就由 AI 驱动、核心价值直接建立在推理/生成/决策上，几个人的小团队产出不一定低于大公司。判断轴是三条：任务复杂度与可靠性/一致性要求、成本是否合适、把 Agent 当 autopilot 还是 copilot。这与 [[concepts/when-not-to-harness-engineering|何时不要 Harness]] 的边界条件互补——那一页从模型代际/成本/安全切入，这一篇从任务形态与团队形态切入。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**四支柱与六层的对应关系**：约束动作空间（工具 / skill / 权限 / 重试熔断）、管理 I/O 与上下文（分层降维、成本）、调度与触发（"何时干活"交给确定性系统）、状态与记忆持久化（记忆、git 版本）。一句话总结：「**Agent = 模型 + Harness，模型负责聪明，Harness 负责稳**」。作者还给了一个很有说服力的观察：公司内部 Agent 大赛近三周，跑通 Demo 很容易，「让模型具备良好的视力和行动能力」不难，**让模型持续稳定输出**才占掉大部分时间。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**技能供给：原子化能力，但不要囤技能**。Agent 一般跑在沙箱里，第一步是让它"看得见、够得着"真实数据——小数据量可以手工搬运，重复性任务要靠稳定的 skill 与 MCP 工具固化（如 idp-mcp 查 Hive、ES 分析即时流量、威胁情报查 IP）；固化后的工具还能提升缓存命中率、降低成本。反面是**技能越多越好是错的**：社区有开发者装了上万个 skill，Agent 效果反而变差——应按解决问题为导向增删迭代。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**上下文管理的第一手段是数据预计算，而不是压缩**：不把低信息密度的数据直接交给模型。数仓三层里 ods 与 dwd 都不适合直接喂模型（各为 T 级），到 **dws 层天级数据只有百 M 级**，再按小时分区就降到几 M，模型只处理几 M 才现实；但 dws 的细粒度分析仍需回到 dwd 明细。配合两类控制：**被动限制**（MCP 返回行数、会话连接时长、思考轮次与时长、token plan）与**主动介入**（在技能文档里规定时间范围/查询范围/查询深度；按优先级层次分析——高风险全量、中风险 TopN、低风险抽样）。具体做法是：先用廉价资源在 dwd 层把流量预计算成风险等级，再让高风险走 ES 全量分析、中低风险做层次分析。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**指令系统的分寸**：系统层注入提示词（`CLAUDE.md` 与 `./prompts`）一次定义好、少改——**频繁调整反而让输出不稳定**。`duties.md` 定义工作模式（收到问题→判断是否需要调工具→制定分析计划→执行工具调用→基于结果给结构化回答）、内容呈现格式、工作原则（用什么工具、方案对比、信息来源）与禁止事项；`rules.md` 管两类边界：禁止中间文件乱放（保持工作空间干净有序）、以及"不自作聪明、未被 @ 时保持沉默"——作者明确说**幻觉靠写记忆压不住，边界性约束必须在 rules.md 里特别声明**。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**调度：把"何时干活"从 Agent 手里拿走**。cron 定时触发在实际生产中问题很大——上游任务完成时间不确定，定时会造成重复执行、漏执行、超时，整体成功率很低；团队试过 A2A 调用（当时不支持）与数据探针，只能解决一部分；最终改为**事件触发**：上游完成后用事件唤起模型，模型不再需要猜上游何时就绪。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**知识管理两条**：①**记忆是长期资产**——人在这里更多承担 reviewer，大量带个人色彩的业务判断、使用偏好、处理流程是私域信息，不前置沉淀就会拉长"磨合期"；②**用 git 管 agent 的基础文件**（系统提示词、记忆文件、启动脚本），保证迭代可回溯、团队可共享、沙箱重建不丢。这正好补上框架层常被忽略的一条：**重建一个数字分身很容易，保证行为一致性很难**。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**运行指标对比（同模型 qwen3.7-plus、同时段）与四项调整**：①调度改为事件管线驱动、不依赖 cron，Agent 不再管理与修改自身调度状态，减少工具调用步骤；②**重试熔断**——每个操作最多重试 2 次、不要反复换参数格式（此前大量 token 与时间花在解决工具调用失败上）；③**查询方式改为一次性导出数据快照到工作空间、后续本地聚合**，消除重复 Hive 读写与错误重试；④**逻辑固化为脚本**——一致性要求高的输出固化为稳定脚本与命令，避免口径不一致、省掉现场推理。效果是 Agent Loop 显著缩短、稳定性提高、执行效率明显改善；整体 token 消耗增加，但增量大多来自**缓存读取与缓存创建**，而**缓存命中价格只有普通输入价格的 1/10**——用少量成本换效率是划得来的。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**尚未解决**：Agent 一味输出而收不到人类反馈，会不断自我强化、陷进"死胡同"出不来；如何收集人类真实反馈与纠正、并让 Agent 总结避免再次踩坑，是下一步的重点。这条与 [[concepts/agent-self-improvement-loops|Agent 自改进循环]] 的"反馈来源必须外部化"同向。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

## 相关页面
- [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]] — Harness 哲学的另一种表达
- [[entities/openclaw-prompt-context-harness|OpenClaw Prompt/Context/Harness]] — OpenClaw 的 Harness 实践
- [[entities/claude-code-prompt-context-harness|Claude Code Prompt/Context/Harness]] — Claude Code 的 Harness 实践
- [[entities/agentcore-harness|AgentCore Managed Harness]] — AWS 托管 Harness 平台
- [[entities/skill-design-patterns|Skill 设计模式]] — Skills 是 Context Engineering 的典型实践
## 相关实体
- [[entities/agent-principle-architecture-engineering-practice|你不知道的 Agent 原理架构与工程实践]]
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2|Harness Engineering: 让 Coding Agent 可靠完成长程任务]]
- [[entities/agent-reliability-context-drift-tool-hallucination|Agent Reliability: Context Drift & Tool Calling Hallucination]]
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering：让 Coding Agent 可靠完成长程任务]]
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 解析：智能体架构深度拆解]]
- [[entities/ai-native-时代-研发组织何去何从|AI Native 时代 —— 研发组织何去何从]]
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|长周期 Agent 详解：从 Ralph Loop 到可接管 Harness]]
- [[queries/harness-peer-review-framework|Harness Design Peer Review Framework]]

- [[entities/agent-era-architect-skills-guide|Agent 时代架构师技能指南]]
- [[entities/design-patterns-for-ai-agents-2026|Design Patterns for AI Agents 2026]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]
- [[entities/martin-fowler-ai-rd-harness-nondeterminism|Martin Fowler AI 研发 Harness：非确定性承重层]]

## 新增关联实体
- [[entities/didi-ibg-customer-experience-llm-quality-inspection-3-pipelines]]
- [[entities/gpt-image-2神级提示词分享]]

## 关联实体

**上游依赖**:
- [[entities/thin-harness-fat-skills]] — 提供基础理论/方法
- [[entities/openclaw-prompt-context-harness]] — 提供基础理论/方法
- [[entities/claude-code-prompt-context-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/skill-design-patterns]] — 具体应用场景
- [[entities/agent-principle-architecture-engineering-practice]] — 具体应用场景
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2]] — 具体应用场景

### Supplementary: 北京大学 Harness Engineering 课件（2026-07）
北京大学 AI 肖睿团队（李晴）发布的 82 页 Harness Engineering 培训课件，从 Prompt→Context→Harness→Loop 四阶段系统梳理演进路径，提出 Harness 7 大问题和 ETCLOVG 七层架构。 ^[raw/articles/peking-university-harness-engineering-taming-agent-2026.md]

**平行协作**:
- [[entities/ai-native-时代-研发组织何去何从]] — 替代/补充方案
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei]] — 替代/补充方案
- [[entities/agent-era-architect-skills-guide]] — 替代/补充方案

- 母体实体：[[entities/harness-engineering-systematic-explainer]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-engineering-exploration-tencent-tech]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-architecture-design-production-guide]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-engineering-jk-launcher-baijiajie]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-engineering-core-patterns]]（嵌入近邻锚点，提案卡 #12 批1）

## 所属 MOC

- [[moc/amazon-aws-ai|Amazon Aws Ai]]
- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
