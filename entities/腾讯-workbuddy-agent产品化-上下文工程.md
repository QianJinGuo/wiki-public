---
title: "腾讯 WorkBuddy 实践：如何把 Agent 做成可用产品"
created: 2026-08-15
updated: 2026-10-21
type: entity
tags: [ai, agent, harness, context-engineering, memory, loop-engineering, mcp, skill, 腾讯]
sources: [raw/articles/腾讯workbuddy实践如何把agent做成可用产品]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 腾讯 WorkBuddy 实践：如何把 Agent 做成可用产品

腾讯 WorkBuddy 策略产品经理 Anne 从产品视角拆解 Agent 的运行机制，沿「模型—上下文—Harness—Loop」链路讲清如何把 Agent 从模型能力推进到可用产品能力。常见判断是「模型够强，剩下交给提示词」，但把 Agent 做成能在生产环境稳定完成任务的产品后会发现，模型只承担其中一部分：工具接入、上下文组织、权限边界、结果验证、反馈纠正和跨会话延续都会直接影响产品是否可靠。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

对产品侧而言，模型可以抽象为「根据输入产生后续文字的函数」，这个抽象包含两条约束，决定了上层所有工程的存在理由：模型是无状态的（产品在模型外部保存对话历史、Memory、数据库，需要时再注入），模型的知识截止到训练日期（实时信息需先用工具查询再放进上下文）。工具调用是模型与外部系统的结构化协议：模型负责生成调用请求，Agent 负责执行——持有 API Key、发起请求、修改数据的是 Agent 不是模型，因此权限、审批、参数校验和审计日志都必须由模型外部的工程机制执行。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## 四个概念：工具调用 / MCP / Skill / Plugin

工具调用解决「一个模型怎么请求执行动作」；MCP 解决「外部系统怎么标准化接入」——Anthropic 2024 年底发布的开放协议，提供 Resources（应用/Agent 驱动的只读内容）、Tools（模型驱动的动作）、Prompts（用户驱动的提示模板）三种原语，2026 年的 MCP Apps 扩展还允许工具返回可直接渲染的交互式 UI；Skill 解决「一类任务应该按什么方法做」——沉淀经过验证的工作方法（说明、步骤、脚本、命令和判断标准），Tool 负责「一个动作」，Skill 负责「一类任务的做法」；Plugin 解决「能力组合如何安装和分发」——把 MCP、Skills、Rules、Hooks、模板组合成可安装分发的单位（MCP 是跨产品协议，Plugin 是产品层的打包概念）。选择外接能力形态的判断标准是：能力边界、更新频率、权限风险、上下文成本、执行延迟与跨产品复用价值。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

一次完整任务（如「调研 OpenAI/Anthropic/LangChain 的 Harness 实践并输出大纲」）的流程是：检查已有资料 → 读取 Memory 和 Skill → 查询内部问题 → 分配 Sub-agent 并行调研 → 汇总观点和证据 → 补充缺口 → 生成大纲，整个过程由多轮「工具调用—拿到工具结果—决定下一步」的 ReAct 循环构成，主上下文长度逐渐增大，因此必须做上下文管理。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## Context Engineering：模型这一刻该看到什么

Context Engineering 定义为：在一次模型决策前，设计哪些信息进入上下文、以什么形式进入、放在什么位置、何时更新或移出，以提高模型做出正确下一步决策的概率。它包含五类动作：写入（把目标、规则、环境和任务状态显式写进上下文）、选择（从已有候选里只挑当前需要的）、检索（不在手的信息按需捞进来）、压缩（长内容外置到文件、只留结论与证据位置）、隔离（用独立会话或 Sub-agent 处理旁支任务只带回结果）。Context Engineering 追求相关、准确、及时，不是单纯堆 token。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

Prompt Cache 是上下文管理的第一要义：按前缀匹配缓存，WorkBuddy 遵循几条规则——System Prompt、基础工具定义、长期规则放前面保持稳定；对话历史追加保存不修改已发送消息；动态内容追加到后面；工具和 Skill 按需加载；只在需要压缩或纠正错误时才接受前缀变化。工具定义与 Skill 都采用渐进式加载（先看名称和简介，进入任务再加载完整内容），意图识别负责「先选对方向」，渐进式加载负责「再按需展开」。工具结果过长时分页、截断或写入文件，截断时明确告诉模型「结果未完整」并附总量、截断位置和继续读取方法。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## Memory：让正确的过去在正确的时候重现

记忆解决的是重复交代背景的问题。WorkBuddy 把长期信息拆成五类陈述性记忆：稳定事实、用户知识背景、行为信号、表达偏好、会话延续信息，每类影响范围不同（Semantic 可作为默认前提、Style 只影响表达、Behavior Signal 需观察多次稳定行为并允许用户纠正）。关键设计是没有把 Procedural Memory（程序性记忆）纳入长期记忆——「做事方法」一旦注入会直接影响推理路径：局部经验可能被误升为通用策略、干扰模型推理、隐性改写 Agent 行为且缺少版本评测审批回滚。WorkBuddy 的选择是用户事实和历史状态放进 Memory，经过验证的工作方法保存为 Skill（可版本化、可评审、可测试、可回滚、按需加载）。记忆按作用域分层（当前轮临时上下文 → 会话/Thread → 项目/Workspace → 用户级 → 团队/组织级），作用范围越大写入晋升门槛越高。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## Harness Engineering：引导、约束与整合

从词源（套在马身上的整套装备）出发，Harness 拆出三类能力：驾驭（Steer，控制执行方向、速度和停止时机——System Prompt、规则文件、Skills、Task/Todo、自我纠正提示）、约束（Constrain，防止执行超出安全范围——权限边界、Sandbox、Approval Gate、Allowlist/denylist、测试验证、rollback、audit log）、整合（Integrate，把执行能力、状态承载、协作机制、自动化配齐并协同）。三类能力必须共同工作：只有引导没有约束，Agent 可能执行不该执行的动作；只有约束没有反馈，出错后无法修正；工具多但缺编排，长任务难以稳定完成。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

业界实践参照：OpenAI 一个 3 人小组用 Codex 从空仓库开发 5 个月产出约 100 万行代码、1500 个 PR；Anthropic 长任务 harness 用「初始化 Agent + Coding Agent」跨会话交接、Planner/Generator/Evaluator 三角色对抗评估；LangChain 定义更宽——Agent = Model + Harness，用 Compaction、Tool Call Offloading、Skills 渐进式加载应对 Context Rot，用 Ralph Loop 拦截提前结束信号。WorkBuddy 构建的 Harness 是控制系统：行动前通过前馈（Feedforward）提供目标、规则、环境和可用能力，行动后通过反馈传感器（Feedback sensors）观察结果并返回修正信息；控制按执行方式分为计算型（LSP、类型检查、linter、单测，快便宜可重复）和推断型（Review Agent、AI judge，覆盖难以写成规则的问题）——能用计算型信号解决的优先交给确定性程序。五层结构：运行环境层、引导层（Feedforward）、反馈层（Feedback，含编辑前时间戳校验、外部验证信号、Audit log）、编排层（渐进式加载、意图识别路由、多模型路由、Teams 协作）、迭代层（Harness 自身随模型能力、用户场景和已发现问题持续调整，需证据支撑并评估副作用）。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## Loop Engineering 与未解决问题

Loop Engineering 关注任务如何被触发、连续执行、验证结果、记录进度并再次运行——工程对象从单条 Prompt 扩展到可长期稳定运行的任务循环。一个可用的 Loop 至少需要：触发器、独立执行环境、Skills、Tools/Connectors/MCP、Sub-agents、Memory/Durable Artifacts、Sensors/Evals、Stop Conditions/Budget。需要强调的是 Goal ≠ Loop——Goal 只定义「要去哪里」，Loop 还需要触发器、执行环境、工具、验证信号和停止条件。Loop 不会自动产生正确目标、不会自动产生可信的验收标准、不会承担责任。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

未解决的问题包括：功能和业务正确性验证缺口（实现和测试可能共享同一个误解，业务正确性缺少可计算判定标准，AI 自治度需随风险提高而降低：核心业务逻辑支付/风控/订单自治度上限低）、代码库的 Harnessability 决定建设难度（老系统缺结构、历史违例多、可观测性弱）、案例有适用边界、Harness 需要持续投入（工程严谨度从代码编写本身部分转移到环境、反馈回路和控制系统的设计上）。两个核心结论：模型决定能力上限，上下文和 Harness 决定这个上限能否稳定落地；人负责选择方向、定义标准并承担责任，Agent 负责执行、验证和加速迭代。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

## 深度分析

### 计算型与推断型：不是分类学，而是一条成本／覆盖率边界

把控制分成计算型（LSP、类型检查、lint、单测、codemod）和推断型（Review Agent、AI judge）看似是分类，实质是一条由两件事定价的边界：判定标准能否被形式化，以及判断错误的代价。计算型快、便宜、可重复，但只能表达已经被写成规则的知识；推断型能覆盖「是否误解需求」「是否过度设计」，代价是慢、贵、不稳定。边界画错的两个方向都会付费：把类型检查能解决的问题交给审查 Agent，是用成本换覆盖率却什么也没换到；把语义判断硬塞进 lint，只会得到一批被静音的通配符规则。

### 只有前馈会自信地漂移，只有反馈会原地打转

前馈给出目标、规则、环境与可用能力，反馈给出结果与修正。缺一的失败模式截然不同：缺反馈时，规则是否有效从未被验证，Agent 会以同样的确信反复产出同一类错误，且在进入人工审查前不会被发现——这是自信的漂移；缺前馈时，Agent 每一轮都要靠事后纠正重新发现约束，同一个坑反复踩，预算烧在返工上——这是抖动。产品上这两类故障需要完全不同的投入：前者补传感器，后者补规则与文档，用错方向只会让症状换个形态出现。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]

### 无状态模型与产品级连续性：状态究竟存在哪里

模型不携带会话之外的状态，而产品承诺的是「上次做到哪、为什么这么做」的连续性。承载物有三类，失败模式各不相同：Memory 让正确的过去在正确的时候重现，但会漂移、被污染、会过期；Durable Artifacts（文件、任务树、changeset）可审计，却容易与真实意图脱节；会话日志能回放，但成本高、检索难。选择本质是在「便宜但会错」与「贵但可查」之间定价——[[concepts/agent-memory-architecture|Agent 记忆架构]]的取舍正沿这条线展开。

### Goal ≠ Loop：停止条件与验收标准归谁所有

Goal 只说明去哪里；Loop 还要求触发器、执行环境、验证信号、预算与停止条件。这句话在组织上的真正含义是所有权：写下停止条件的人，为「什么时候放弃」负责；定义验收标准的人，为「什么算完成」负责。多数失败的 Loop 不是跑不起来，而是这两样东西没人拥有——于是它要么永不停止地烧预算，要么在无人认可的标准上宣布成功。[[entities/loop-engineering-feedback-control-system|反馈控制系统]]提供了机制，但不提供责任人。

### Harnessability 是上游约束，Prompt 补不回来

老系统缺结构、历史违例多、可观测性弱，这类约束位于所有 Prompt 工作之前：没有类型与测试，计算型反馈无从产生；没有埋点与链路追踪，运行时信号只能退化为模型自评。在低 Harnessability 的库上，「换更强模型」与「把规则写得更细」的边际收益都接近于零。投入顺序必须先补可验证性（测试、类型、结构化入口），再提自治度；反过来做，等于在流沙上盖控制室，这正是[[concepts/when-not-to-harness-engineering|什么时候不该做 Harness]]所警示的。

## 实践启示

1. 按「能否形式化」给反馈信号排预算：类型检查、lint、单测放在每次编辑后免费反复跑；架构审查、AI judge 只在提交前与高风险变更加跑。一个信号如果每次运行都让人想静音，它该被左移或被删除。
2. Audit log 至少记录四样：动作、目标文件的时间戳校验结果、触发它的信号、当时的上下文摘要。目的不是合规，而是让「Agent 为什么这样改」事后可重放，否则纠错只能靠整段重跑。
3. 按风险分层封顶自治度：支付、风控、订单等核心路径限制为「提议 + 人工确认」，外围脚本与文档可以全自动。上限要写成配置而非写进 Prompt，Prompt 里的边界在长会话中必然衰减。
4. 上线一个 Loop 前先写死三件事：停止条件（轮数／预算／连续无进展）、验收标准（谁签字、看哪个信号）、失败报告落在哪里。缺一件，Loop 就会以「还在跑」的形式长期掩盖失败。^[raw/articles/腾讯workbuddy实践如何把agent做成可用产品.md]
5. 用「首次正确率 + 人工介入率」两条曲线判断 Harness 投入是否回本，而不是数规则条数。规则变多而介入率不降，说明补的是症状；介入率降但返工轮次上升，说明约束过紧，正在把成本推给下游。
6. 每个季度回收一次失效机制：模型升级后，不少为旧模型弱点写的规则与 Skill 会从「护栏」退化为「噪音」。[[concepts/harness-engineering|Harness Engineering]]是持续运行的控制系统，不是一次性配置。

## 相关实体

- [[concepts/context-engineering|上下文工程]]
- [[concepts/harness-as-product-surface|Harness 即产品面]]
- [[entities/workbuddy-product-framework-agent-harness-anne-2026|WorkBuddy 产品框架]]
- [[entities/workbuddy-skill-全拆解从创建到自进化|WorkBuddy Skill 全拆解]]
- [[concepts/harness-loop-architecture|Harness Loop 架构]]
- [[concepts/context-management-agent-systems|上下文管理系统]]

→ [[raw/articles/腾讯workbuddy实践如何把agent做成可用产品|原文存档]]
