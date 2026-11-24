---

title: "Agent 工程原理、架构与实践"
type: entity
tags: [agent, architecture, harness]
created: 2026-05-21
updated: 2026-11-22
review_value: 7
review_confidence: 7
sources: [raw/articles/agent-engineering-principles-architecture-practice]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# agent-engineering-principles-architecture-practice

你不知道的 Agent：原理、架构与工程实践 ^[raw/articles/agent-engineering-principles-architecture-practice.md]
文章内容基于作者个人技术实践与独立思考，旨在分享经验，仅代表个人观点。 ^[raw/articles/agent-engineering-principles-architecture-practice.md]
这篇文章主要讲 Agent 架构里几块最影响工程效果的内容，包括控制流、上下文工程、工具设计、记忆、多 Agent 组织、评测、追踪和安全，最后再用 OpenClaw 的实现把这些设计原则串起来看一遍。 ^[raw/articles/agent-engineering-principles-architecture-practice.md]
整理下来，有几处判断和我原来想的不太一样，更贵的模型带来的提升，很多时候没有想象中那么大，反而 Harness 和验证测试质量对成功率的影响更大，调试 Agent 行为时，也应优先检查工具定义，因为多数工具选择错误都出在描述不准确，另外，评测系统本身的问题，很多时候比 Agent 出问题更难发现，如果一直在 Agent 代码上反复调，效果未必明显，读完这篇，这几个问题应该能有些答案。 ^[raw/articles/agent-engineering-principles-architecture-practice.md]
Agent Loop 的核心实现逻辑抽象后其实不到 20 行代码： ^[raw/articles/agent-engineering-principles-architecture-practice.md]

## 深度分析

### 一、控制流与 Harness：循环本身稳定，外围工程决定成败

Agent Loop 的核心实现抽象后不到 20 行代码：while 循环里调用模型、按 stop_reason 判断是否为 tool_use、执行工具、回填 assistant 与 tool_result 消息，直到模型返回纯文本退出。感知→决策→行动→反馈的循环形态在各实现与官方 SDK 中高度一致，说明循环本身相当稳定，从最小实现扩展到复杂系统不需要推翻控制流，新能力只通过扩展工具集和 handler、调整系统提示结构、状态外化三种方式接入 ^[raw/articles/agent-engineering-principles-architecture-practice.md:19-47]。

Workflow 与 Agent 的边界由执行路径决定：路径由代码预先写死的是 Workflow，由 LLM 动态决定的是 Agent ^[raw/articles/agent-engineering-principles-architecture-practice.md:48-49]。大多数 AI 系统拆开看都是五种控制模式的组合——提示链、路由、并行化、评估-优化、orchestrator-worker，工程问题是选对组合而非发明新模式 ^[raw/articles/agent-engineering-principles-architecture-practice.md:50-51]。

Harness 是围绕 Agent 构建的测试、验证与约束基础设施：验收基线、执行边界、反馈信号与回退手段。核心判断是模型虽然重要，但决定系统能否稳定运行的往往是这些外围工程条件；任务清晰度和验证自动化程度共同决定 Agent 的适合区域，目标明确且结果可自动验证的右上角区域最适合部署 ^[raw/articles/agent-engineering-principles-architecture-practice.md:52-55]。这解释了引言中"更贵的模型提升不如预期"的现象：瓶颈常在 Harness 而非模型，投入转向 Harness 建设（见 [[entities/harness-engineering-framework|Harness 工程框架]]）边际收益更高。

### 二、上下文工程与工具设计：分层防稀释，ACI 定接口

上下文工程决定稳定性的根源在注意力机制：Transformer 注意力复杂度 O(n²)，上下文越长关键信号越容易被噪声稀释，即 Context Rot ^[raw/articles/agent-engineering-principles-architecture-practice.md:57]。对策是五层分层：常驻层（身份定义、项目约定、绝对禁止项，短硬可执行）、按需加载（Skills 与领域知识，描述符常驻、完整内容触发时注入）、运行时注入（时间、渠道、偏好等动态信息）、记忆层（跨会话经验写 MEMORY.md）、系统层（Hooks 处理确定性逻辑，不进上下文）^[raw/articles/agent-engineering-principles-architecture-practice.md:58-63]。量化结论支撑分层：常驻层越稳定 Prompt Caching 前缀命中率越高；Skills 按需加载有明确反例——没有反例时准确率从 73% 掉到 53%；压缩保留优先级依次为架构决策、已修改文件、验证状态、未解决问题 ^[raw/articles/agent-engineering-principles-architecture-practice.md:64-66]。

工具侧同构：定义质量比数量更关键，仅 5 个 MCP 服务器就可能带来约 55,000 tokens 的工具定义开销 ^[raw/articles/agent-engineering-principles-architecture-practice.md:68]。设计演进三代：API 封装（粒度过细）→ ACI（工具对应 Agent 目标而非底层 API）→ Advanced Tool Use（Tool Search + Programmatic Tool Calling + Examples）^[raw/articles/agent-engineering-principles-architecture-practice.md:69-72]。ACI 三原则是面向目标、边界明确、参数防错、定义里直接给示例；调试 Agent 应先检查工具定义，多数工具选择错误出在描述不准确而非模型能力 ^[raw/articles/agent-engineering-principles-architecture-practice.md:73-74]。这与 [[concepts/context-engineering|上下文工程]] 是一枚硬币的两面：上下文决定模型看得见什么，工具定义决定模型够得着什么。

### 三、记忆、多 Agent 与自主度：状态外化是共同底座

记忆采用四级分类：上下文窗口即工作记忆、Skills 即程序性记忆、JSONL 会话历史即情景记忆、MEMORY.md 即语义记忆 ^[raw/articles/agent-engineering-principles-architecture-practice.md:76-80]。OpenClaw 的实现以 memory/YYYY-MM-DD.md 追加写加 MEMORY.md 精选，检索为 70% 向量 + 30% 关键词混合；整合由 tokenUsage/maxTokens ≥ 0.5 阈值触发，采用可回退设计——只移动指针、不删除原始消息，避免整合出错时信息丢失 ^[raw/articles/agent-engineering-principles-architecture-practice.md:82-83]。

长任务跨 session 靠状态外化：Initializer Agent 首轮生成 feature-list.json、init.sh 与 git commit，Coding Agent 接续执行；任务状态显式写成 JSON 而非 Markdown，保证机器可解析 ^[raw/articles/agent-engineering-principles-architecture-practice.md:84-86]。多 Agent 分指挥者（同步协作，session 结束上下文即丢失）与统筹者（异步委派，产出为分支/PR 等可持久化工件）两种模式；协作协议核心是 JSONL inbox + Worktree 隔离 + 任务图依赖，子 Agent 只回传摘要、细节留在自己的消息历史里 ^[raw/articles/agent-engineering-principles-architecture-practice.md:88-92]。关键约束是幻觉会互相放大，引入顺序必须是先有可持久化任务图、再引入有身份的队友、再引入结构化协议，最后才放开并行 ^[raw/articles/agent-engineering-principles-architecture-practice.md:93]。

### 四、评测、追踪与落地：先修评测，再改 Agent

评测坐标系由三组概念构成：task/trial/grader（测什么、跑多少次、怎么打分）、transcript/outcome（执行记录与环境最终结果）、agent harness 与 evaluation harness 分离 ^[raw/articles/agent-engineering-principles-architecture-practice.md:95-98]。两个指标各管一段：Pass@k 衡量 k 次至少一次正确的探索上限，Pass^k 要求 k 次全部正确，才是上线回归标准；评分器按确定性从高到低选：代码 > 模型 > 人工 ^[raw/articles/agent-engineering-principles-architecture-practice.md:100-102]。最有工程价值的判断是"先修评测，再改 Agent"：评测系统的出错来源（资源不足、评分器 bug、用例脱节、只看聚合分数）与模型退化表现一模一样，反复调 Agent 而效果不明显时问题大概率在评测侧 ^[raw/articles/agent-engineering-principles-architecture-practice.md:103]。

追踪是排查前提：Trace 记录完整 Prompt、messages、工具调用参数与返回值、推理链、最终输出及 token 消耗和延迟；可观测性分人工抽样标注与 LLM 全量自动评估两层；采样策略包括负反馈 100% 进队列、高成本对话优先审查、随机采样、变更后 48 小时全量；事件流以 tool_start/tool_end/turn_end 三节点 emit，一次发布多路消费 ^[raw/articles/agent-engineering-principles-architecture-practice.md:105-108]。

OpenClaw 落地验证了闭环：五层架构（Gateway、23+ 渠道适配器、Pi Agent 主循环、ACI 工具集、记忆），MessageBus 解耦渠道与 Agent（dispatch 不做 await，不同 session 并发、同一 session 串行），安全边界先于功能（白名单授权、realpath+relative 隔离检查、审计日志），加 Prompt Injection 防护与 Provider 故障切换两层兜底 ^[raw/articles/agent-engineering-principles-architecture-practice.md:109-113]。反模式清单同样重要——系统提示当知识库、工具数量失控、缺验证、多 Agent 无边界、记忆不整合、无评测、过早引入多 Agent，都是跳过上述原则的典型后果 ^[raw/articles/agent-engineering-principles-architecture-practice.md:114-115]。安全设计可对照 [[concepts/ai-safety|AI 安全]]，记忆实现细节见 [[entities/agent-memory-architecture|Agent 记忆架构]]。

## 实践启示

1. **构建 Harness 验证基础设施**：优先于模型选型，建立自动化验收基线和执行边界，让对错有机器可执行的判断标准 ^[raw/articles/agent-engineering-principles-architecture-practice.md]

2. **用五层架构管理上下文风险**：常驻层保持短硬可执行，Skills 按需加载只保留索引，完整知识触发时再注入 ^[raw/articles/agent-engineering-principles-architecture-practice.md]

3. **按 ACI 原则设计工具**：面向目标而非 API、边界明确、参数防错、定义里直接给示例，避免工具数量失控 ^[raw/articles/agent-engineering-principles-architecture-practice.md]

4. **建立 MEMORY.md 加可回退的记忆系统**：混合检索（70% 向量 + 30% 关键词），token 阈值触发整合，指针移动不删除原始消息 ^[raw/articles/agent-engineering-principles-architecture-practice.md]

5. **先建评测体系再优化 Agent**：区分 Pass@k（探索能力上限）和 Pass^k（上线回归），按确定性选择评分器（代码 > 模型 > 人工） ^[raw/articles/agent-engineering-principles-architecture-practice.md]

## 相关实体
- [[entities/agent-principle-architecture-engineering-practice]]
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2]]
- [[entities/factory-mission-multi-agent-architecture]]
- [[entities/harness-engineering-long-term-agent-tasks]]
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent]]
- [[moc/wiki-master-map|MOC]]

→ [[raw/articles/agent-engineering-principles-architecture-practice|原文存档]] ^[raw/articles/agent-engineering-principles-architecture-practice.md]
