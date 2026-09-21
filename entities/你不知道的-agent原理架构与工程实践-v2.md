---

title: "你不知道的 Agent：原理、架构与工程实践"
created: 2026-06-10
updated: 2026-09-21
tags: [agent, architecture, code, data, database, evaluation, llm, memory, mlops, prompt, security, tool-use]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/你不知道的-agent原理架构与工程实践-v2
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 你不知道的 Agent：原理、架构与工程实践

→ [[raw/articles/你不知道的-agent原理架构与工程实践-v2|原文存档]]^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

## 深度分析

### Agent Loop 不变，Harness 决定收敛

Agent Loop 抽掉工程细节后不到 20 行：一个 messages 数组加一个 while(true)，模型返回 tool_use 就并行执行工具、把 tool_result 追加回历史，返回纯文本即结束，对应控制流是感知 → 决策 → 行动 → 反馈的持续循环。作者的观察是循环本身极稳定，从最小实现扩到子 Agent、上下文压缩和 Skills 加载，主循环几乎没变，能力只从三个地方长出来——扩展工具集与 handler、调整系统提示结构、把状态外化到文件或数据库；模型负责推理，外部系统负责状态与边界。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

Anthropic 划分 Workflow 与 Agent 的标准是控制权：路径由代码写死是 Workflow，下一步由 LLM 动态决定才是 Agent，现实中不少标着 Agent 的产品深看更接近 Workflow。文章因此列出五种常见控制模式——提示链、路由、并行（分段法与投票法）、编排器-工作者、评估器-优化器，并强调多数系统只是它们的组合，很多场景并不需要完整自主权。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

Harness 才是分水岭。这里把 Harness 限定为围绕 Agent 的验收基线、执行边界、反馈信号和回退手段，并用「任务清晰度 × 验证自动化程度」四象限说明：只有目标明确且结果可自动验证的右上角是 Agent 的甜区，任务清楚但要人盯的左上角吞吐量受限于人的审查速度，有自动反馈但目标模糊的右下角会让系统高效地跑向错误方向，Harness 的职责就是把任务推进右上角。这个判断在高可验证任务（写代码）上最成立，在开放式研究、多轮协商这类弱验证任务上模型上限仍更关键。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

OpenAI 的 agent-first 实践是这组判断的实证：3 个工程师 5 个月产出近 1500 个 PR、百万行代码，约为传统速度的 10 倍，靠的不是更强模型，而是四条工程决策——Agent 看不到的内容等于不存在（知识必须落在代码库里，AGENTS.md 只留约 100 行做索引）、约束编码化而非文档化（架构分层由自定义 Linter 机械强制）、端到端自主（连查日志、指标、追踪都由 Agent 主动完成）、最小化合并阻力。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 上下文工程：分层、压缩与缓存前缀

失效模式往往不是窗口不够长，而是信息密度不对：无关内容一旦占到大头，决策质量就明显下滑，即 Context Rot，很多看似模型能力不足的问题可以追溯到上下文组织。解法按使用频率与稳定性分层——常驻层（身份、项目约定、绝对禁止项，短、硬、可执行）、按需加载（Skills 描述符常驻、完整知识触发时才注入）、运行时注入（时间、渠道 ID、用户偏好）、记忆层（写入 MEMORY.md，需要时再读）、系统层（Hooks 或代码规则，完全不进上下文），并贯彻一条原则：能用 Hooks、代码规则或工具约束表达的确定性逻辑，不要放进上下文。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

三种压缩策略取舍不同：滑动窗口成本极低但丢早期上下文；LLM 摘要成本中等、丢细节留决策，进阶做法 branch summarization 会在摘要时明确保留架构决策、未完成任务和关键约束；工具结果替换成本极低，靠 micro_compact 每轮替换旧输出、auto_compact 超阈值时触发。真正容易出错的是保留顺序，文章建议把优先级写进 CLAUDE.md 一类文档——架构决策不得摘要、已修改文件与关键变更、验证状态、未解决 TODO 与回滚笔记、工具输出可删只留 pass/fail 结论；同时 UUID、hash、IP、端口、URL、文件名等标识符必须原样保留，改错一位 PR 编号后续工具调用即失效。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

Prompt Caching 的底层是 Transformer 为每个 token 计算 KV 对，只有精确前缀匹配才命中，所以缓存友好设计的核心是稳定性：系统提示、工具定义、长文档放前面，动态信息放后面；这也说明「常驻层短而稳定」不只为了省 token，还在保护前缀命中，而稳定的大系统提示比频繁变动的小提示实际更便宜，因为写入成本只付一次、后续读取折扣可达 90%。Skills 延迟加载同理，按需内容追加在稳定前缀之后不破坏缓存；反过来，接了多个 MCP 工具且工具集频繁变动的 Agent，缓存会持续失效。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

Skills 的写法有硬数据：描述要短（低效写法约 45 tokens，高效写法约 9 tokens，而每个启用的描述符都常驻上下文），要写成路由条件而不是功能介绍（Use when / Don't use when 加反例）——没有反例时准确率从基准 73% 掉到 53%，加上反例升到 85%，响应时间还降 18.1%。只用高频 Skill 进默认列表，低频手动引入，极低频直接用文档；典型反模式是把几百行工作手册塞进 Skill 正文、一个 Skill 覆盖 review/deploy/debug/incident、以及有副作用的 Skill 不显式限制调用时机。文件系统作为上下文接口（Cursor 的 Dynamic Context Discovery）也被验证：工具结果写文件、Agent 用 grep 按需读，把 MCP 工具描述同步到文件夹、默认只暴露工具名，A/B 测试中调用 MCP 工具的任务总 token 消耗减少 46.9%；长任务压缩时把聊天记录完整保留为文件、摘要只引用路径，压缩就从不可恢复的硬截断变成有损但可追溯的操作。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 工具设计：多数选择错误源于描述失真

上下文决定模型能看到什么，工具决定模型能做什么，而工具定义本身就要付费：仅 5 个 MCP 服务器就可能带来约 55,000 tokens 的定义开销，在 200K 上下文里还没开始对话就用掉近三成，工具过多还会稀释模型对单个工具的注意力。好坏工具的差别集中在四件事：粒度对应 Agent 目标而非 API 操作（update_yuque_post 而不是 get_post + update_content + update_title）、返回只给与下一步决策直接相关的字段、错误结构化并带修正建议（而不是一句 "Error"）、描述说明何时用与何时不用。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

工具设计经历过三代：第一代把 API Endpoint 直接封装成工具，粒度过细，Agent 往往要协调多个工具才能完成一个目标；第二代 ACI（Agent-Computer Interface）要求工具对应 Agent 的目标，一次把目标动作说完整；第三代 Advanced Tool Use 进一步优化发现、调用与描述——Tool Search 让 Agent 按需发现工具定义，上下文保留率可达 95%、Opus 4 准确率从 49% 提升到 74%；Programmatic Tool Calling 让模型用代码编排工具调用、中间结果在执行环境里流转不进 LLM 上下文，token 从约 150,000 降到约 2,000；Tool Use Examples 给每个工具附 1-5 个真实示例，弥补 JSON Schema 只能描述参数类型、无法表达调用方式的缺陷，调用准确率从 72% 提升到 90%。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

落地写法上，文章推荐用 betaZodTool 把定义与实现绑在一起、让参数描述直接约束格式、错误用 ToolError 携带 error_code 与 suggestion（例如「文章 ID 不存在 → 请先调用 list_yuque_posts」），让 Agent 更容易一次选对、失败后也能快速修正；由此得出最实用的调试法则：Agent 行为异常时优先检查工具定义，多数工具选择错误出在描述不准而不是模型能力，工具数量也要克制，能用 Shell 处理的、只需静态知识的、更适合 Skill 的都不必新增工具。此外框架内部事件（压缩发生、通知推送、工具调用被跳过）必须隔离——应用层的 AgentMessage 可携带任意自定义字段，真正发给 LLM 的 Message 只保留 user、assistant、tool_result 三种标准类型，会话历史保留完整框架状态，模型只收它需要的部分。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 状态外化与隔离边界：记忆、长任务、多 Agent 的共同答案

Agent 没有原生时间连续性，会话结束上下文即清空，记忆必须作为一层基础设施单独设计。文章按要解决的问题把记忆分成四类：上下文窗口是工作记忆、Skills 是程序性记忆、JSONL 会话历史是情景记忆、MEMORY.md 是语义记忆。产品实现上，ChatGPT 的四层记忆（会话元数据、约 33 条用户偏好事实、约 15 个最近对话摘要、当前滑动窗口）没有向量库也没有 RAG，比很多人预期简洁；OpenClaw 则是 memory/YYYY-MM-DD.md 追加日志 + MEMORY.md 精选事实 + memory_search 的 70% 向量 / 30% 关键词混合检索。作者的判断是：对多数 Agent，结构化 Markdown 加关键词搜索已经足够可调试、可维护、成本可控，只有规模超过几千条且确实需要语义相似度时才引入向量检索。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

整合流程的关键不是摘要写得多漂亮，而是可回退：以 tokenUsage / maxTokens >= 0.5 为阈值，成功路径做 llmSummarize 后追加进 MEMORY.md、只更新 lastConsolidatedIndex，失败路径把原始消息写入 archive/；系统只移动指针、不删除原始消息，整合失败也还能回到存档继续工作。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

长任务最常见的失败不是单步报错，而是 session 结束时任务还没做完——要么在单个 session 里试图做完整个应用导致上下文先耗尽，要么只做了一部分而下一轮无法准确恢复现场。更稳的做法是拆成 Initializer Agent（只跑一次，生成 feature-list.json、init.sh、初始 git commit 与 claude-progress.txt，先把任务变成可持久化的外部状态）和 Coding Agent（每个 session 从 progress 文件与 git log 恢复现场，实现一个功能、跑测试、更新 passes 字段、提交后退出）。要点是进度放文件不放上下文、功能清单用 JSON 便于模型稳定修改、同一时间只允许一个 in_progress、以及把慢速 I/O 放到后台线程并通过通知队列在下一轮 LLM 调用前注入结果——这比把整个 loop 改造成复杂 async runtime 更稳也更好维护。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

多 Agent 的工程顺序是先隔离与协作、再谈并行：指挥者模式是同步协作，人机紧密互动但 session 结束后 context 与产出都很短暂；统筹者模式是异步委派，人只在起点与终点出现，中间产出沉淀为分支、PR 这类可持久化工件，多 Agent 的主要价值正在于把人的持续参与换成对工件的最终审核。落地至少需要三样东西——协议（.team/inbox/{agentId}.jsonl，带 status、append-only、崩溃可恢复）、任务图（.tasks/ 记录依赖）、隔离边界（.worktrees/ 隔离文件修改）；子 Agent 只回摘要，搜索与调试细节留在自己的 messages[] 里，并设有最大深度限制与最小系统提示（只给 Tooling、Workspace、Runtime，不带 Skills 和 Memory）。文章还提示多 Agent 会互相放大幻觉——A 先带偏、B 强化、C 叠加，最后所有 Agent 收敛到同一个高置信度的错误结论，交叉验证能打断这条链，而引入顺序是：先有可持久化任务图，再有有身份的队友，再有结构化通信协议，最后才加交叉验证或外部反馈。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 评测与可观测性：先修评测，再改 Agent

Agent 评测的结构比单轮评测复杂一个层级：要先准备工具、运行环境和任务，Agent 执行中多次调用工具并修改环境，评分看的是跑一批测试验证环境里真正发生了什么，而不是它说了什么。需要记住三组概念——task（测什么）/ trial（跑多少次）/ grader（怎么打分），transcript（完整执行记录）/ outcome（环境最终结果），agent harness（被评测的运行框架）/ evaluation harness（评测基础设施）/ evaluation suite（任务集合）。现状是人工审查与 LLM judge 仍占主导、传统 ML 指标只占 16.9%、近四分之一团队还没开始做评测；指标上 Pass@k 回答「理论上能不能做到」、Pass^k 回答「已有功能有没有被改坏」，混用容易误判，回归测试过松会漏问题，能力评测过严又会让每次小改动都告警。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

评分器按确定性排序是代码评分器（字符串匹配、单测 pass/fail、结构比对、工具参数验证）、模型评分器、人工评分器（可靠但慢，用于建立基准和校准自动 judge），有明确正确答案就优先用代码评分器；而「看 Agent 怎么说」与「看系统最后变成什么样」必须都覆盖——Anthropic《Demystifying evals for AI agents》里的机票预订例子中，Opus 4.5 发现了航空公司政策漏洞、给用户找到更便宜的方案，只按预设路径打分会把这次运行判失败。起步不需要完整体系，20 到 50 个真实失败案例就够，来源优先选已经在人工检查的内容；一个有用的判断标准是：如果两个领域专家拿同一案例独立判断结论不一致，说明验收标准还没写清楚，先解决定义再收数据。环境隔离是最常被忽略的细节，每次运行都从干净状态开始，否则「模型退化」可能只是环境脏了；用例要正例反例都覆盖，通过率接近 100% 时主动补更难的题。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

最容易踩的误区是看到表现下降就立刻改 Agent：评测本身出问题（运行环境资源不足导致进程被杀、评分器 bug 把正确判成失败、用例与生产脱节、只看聚合分数而漏掉某类任务系统性变差）在数字上与模型退化一模一样，文章给的动作顺序是先查环境、再动 Agent，否则改的方向可能从一开始就是错的。可观测性同理，接口层 APM 全绿也可能掩盖某一轮的错误决策，因此 Trace 必须记录完整 Prompt、多轮 messages[]、每次工具调用与参数和返回值、推理链、最终输出与 token 消耗和延迟，并尽量支持语义检索；两层可观测性要一起用——人工抽样标注摸清失败模式并提供校准数据，LLM 自动评估做大范围覆盖；在线评测按规则而非随机采样 10%-20%（负反馈 100% 进队列、超 token 阈值优先审查、每天固定时段随机采、模型或 Prompt 变更后头 48 小时全量）。底座用事件流：在 tool_start、tool_end、turn_end 三个节点发事件，一次发布、多路消费，日志系统、UI 更新、在线评测、人工审查队列各自订阅，主循环不必为任何下游改代码。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 用 OpenClaw 串联设计原则

OpenClaw 把上述原则落成五层解耦：Gateway（WebSocket，端口 18789）统一接住外部连接与系统控制信号，Channel 适配器（23+ 渠道）把渠道差异收敛在 adapter 层，Pi Agent 维护主循环、会话状态、调度与工具调用，工具集按 ACI 原则设计，上下文与记忆层用 Skills 延迟加载加 MEMORY.md 整合（50% token 阈值自动触发）。渠道与 Agent 之间加一层 MessageBus：Channel 只实现 start/stop/send，AgentLoop 从 Bus 消费消息、处理完再发回；dispatch 不做 await，让不同 session 的消息并发处理，但同一 session 必须串行（否则并发写历史和触发 compact 会有竞态，生产环境要对每个 sessionKey 维护队列或 mutex），session 也由 AgentLoop 统一管理而不下沉到渠道层，所以换成 Discord 时 Agent 核心代码不用动。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

系统提示按层叠加：常驻部分由 SOUL.md（身份、核心行为约束、完成标准——「完成等于任务验证通过，且结果已明确反馈给用户」）、AGENTS.md、TOOLS.md、USER.md、MEMORY.md 与 Skills 索引组成，再按当前会话补时间、渠道名、Chat ID 等动态信息；三种触发模式的加载范围不同，普通会话加载完整系统提示，子 Agent 只加载最基础的运行时信息，heartbeat 只加载 HEARTBEAT.md（每 5 分钟轮询待处理任务），长任务超过 20 轮时每轮开头加一行身份重申，用来压住任务漂移。长任务恢复则把 TaskState 写到 .openclaw/tasks/{taskId}.json、每完成一步保存，重启后 resumeTask 从断点继续——超过半小时的任务，崩溃恢复是必选项而不是可选项。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

安全被放在功能之前，三件事先到位：白名单授权（只有授权用户能触发 Agent）、工作空间隔离（shell 工具用 path.relative 做路径检查、用 execFile 而非 exec 避免注入）、audit.jsonl 审计日志（每次执行记时间、用户、命令）。在此之上再补两层兜底：一层按 source-sink 拆解 Prompt Injection——最小权限（不给 Agent 不需要的工具，没有 sink 则 source 侧注入无法落地）、敏感操作显式确认、外部内容用 <untrusted_content> 边界标注来源、关键路径引入独立 LLM 复核；另一层是 Provider fallback，Anthropic 503 或 OpenAI 限速时自动切下一个，不靠人盯。落地顺序同样可复用：单渠道先跑通（不要第一版就抽象多渠道）、安全边界先于功能、记忆整合要早做（不加整合第 20 轮对话后基本就垮了）、Skills 先于新工具、第一个失败就建评测。^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

### 关联实体

- [[entities/harness-之后-状态边界与失败闭环-若飞]]
- [[entities/ai-agent-engineer-learning-roadmap-backend-2026]]
- [[entities/ai-friendly-architecture-design-taobao]]
- [[entities/headroom-context-compression-agent-vibecoder]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/ai-agent-harness-construction-akshay-baoyu]]
- [[concepts/context-engineering|上下文工程]]
- [[concepts/harness-engineering|Harness Engineering]]
- [[concepts/harness-tool-design-evolution|工具设计演进]]
- [[concepts/agent-memory-system-design|Agent 记忆系统设计]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/multi-agent-context-isolation|多 Agent 上下文隔离]]
- [[entities/better-harness-eval-trace-methodology|Eval / Trace 方法论]]
- [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构八段总结]]

## 实践启示

1. **Agent 设计**: 关注控制流与上下文工程的平衡，Harness 约束比模型能力更影响成功率 ^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]
2. **可观测性**: Agent 行为调试应优先检查工具定义和上下文质量 ^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]
3. **渐进式部署**: 从简单 ReAct 循环起步，逐步引入多 Agent 编排 ^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]
4. **验证优先**: 建立完善的测试验证体系，确保 Agent 行为可预测 ^[raw/articles/你不知道的-agent原理架构与工程实践-v2.md]

