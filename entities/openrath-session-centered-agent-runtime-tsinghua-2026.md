---
title: "OpenRath：以 Session 为核心的多 Agent 运行时状态系统（清华）"
created: 2026-06-28
updated: 2026-10-26
type: entity
tags: [agent-runtime, session, state-management, multi-agent, tsinghua, fork-merge, provenance, evidence-protocol, pytorch-analogy]
sources: [raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OpenRath：以 Session 为核心的多 Agent 运行时状态系统（清华）

清华开源的多 Agent 运行时系统，核心创新是将分散在聊天记录、工具日志、沙箱、记忆和分支里的运行状态，统一收进一个可分叉、可合并、可回放的 **Session** 对象。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 核心问题

多 Agent 系统状态散落：聊天记录在消息列表里，工具调用在日志里，沙箱位置在执行器里，记忆在数据库里，分支关系在控制器代码里。没有共同对象把它们串起来。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## Session：不只是聊天记录

Session 是程序真正传来传去的运行时值，包含：对话块、沙箱位置、分支血缘、token 用量、工具证据、记忆交互。可以理解为一次 Agent 工作的"运行账本"。 ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

| 记录类型 | 主要读者 | 主要内容 |
|----------|----------|----------|
| 图检查点 | 调度器 | 执行到哪一步 |
| Trace span | 观察者 | 运行时观察到什么 |
| **Session** | **Agent 程序** | **可传递的工作状态** |

## PyTorch 类比

| PyTorch | OpenRath | 含义 |
|---------|----------|------| ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]
| Tensor | Session | 流动的运行时值 |
| Device | Sandbox | 工具执行的位置 |
| Parameter | Memory | 持久状态平面 |
| Module | Workflow | 可组合工作流 |

核心对象：Session, Agent, Tool, Sandbox, Memory, Workflow, Selector。每个对象"不拥有"什么很重要——Agent 不拥有完整会话图，血缘属于 Session；Tool 不拥有执行位置。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## Session 生命周期

创建→放置→变换→分叉→合并→持久化→释放。

fork 复制当前状态并保留父子关系；detach 切断父血缘；merge 检查沙箱兼容性。分支是否能合并，既看内容，也看执行环境。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 工具执行边界

模型看到 schema → 副作用通过 Sandbox 落地 → 结果作为 Session 证据返回。从"模型说它做过什么"变成"运行时记录它实际做过什么"。

## Claim-to-evidence 协议

每个结论挂证据等级：supported、partially supported、prerequisite-supported、evidence-gated。evidence packet 包含命令、manifest、源代码、Session JSONL、生成产物和证明说明。 ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 与现有框架的区别

| 框架 | 关注点 |
|------|--------|
| AutoGen | 多 Agent 对话可编程 |
| LangGraph | 图状态解决持久执行 |
| OpenAI Agents SDK | agents、handoff、guardrails 产品化 |
| MCP | 工具接入协议边界 |
| **OpenRath** | **运行时状态边界** |

## 局限

- 没有广泛 benchmark 对比
- Memory 质量未验证
- 安全属性未证明（间接 prompt injection 等）
- 云端后端未验证

## 深度分析

### 抽象层级的分野：无状态网关 vs 有状态运行时

大多数所谓 Agent 框架，实质上是无状态网关：请求进来，拼 prompt、调模型、跑工具、返回文本。请求之间唯一的连续性靠外部数据库或聊天历史数组维持，框架本身并不持有"这次工作的状态"。这种设计在单轮或短链路上非常干净，但一旦任务需要分支、复核、回滚，状态的所有权就变成一个没有归属的问题——每个环节都能看见一部分，没有一个环节能回答整体。

OpenRath 的判断是：状态不该由调用方拼装，而该由运行时值承载。原文那句"Session 是程序真正传来传去的运行时值"是关键措辞——它不是存储层的一个主键，而是函数签名上的一个参数。存储层主键只能被读，运行时值却可以被继续变换、被 fork、被 merge。前者是审计对象，后者是计算对象。把证据挂在计算对象上，才让"跑完之后追问这条结论来自哪条分支"成为运行时可以回答的问题，而不是事后从分散日志里考古。这也正是它与 [[concepts/llm-observability-4-layer-model|LLM 可观测性四层模型]] 的分工差别：可观测性回答"外面看到了什么"，Session 回答"里面还剩下什么可以继续用"。

### PyTorch 类比的边界

类比真正的价值在于规定职责边界，不在于精确。Tensor/Session、Device/Sandbox、Parameter/Memory、Module/Workflow 这组映射里，最锋利的一刀是那张"每个对象不拥有什么"的负面清单：Agent 不拥有完整会话图，Tool 不拥有执行位置，Workflow 不额外制造私有调度状态。

但失效点同样清楚。Tensor 是纯值，可交换、可并行、可微；Session 不是。它装着沙箱句柄、token 用量、待处理工作这些带外部副作用和不可逆性的东西，两个 Session 相加没有意义，fork 也不是零成本的视图复制——复制的是状态加血缘。更根本的是，PyTorch 的成功依赖自动微分这种"全局可推导"的属性，而 Agent 运行时没有任何等价物：系统无法自动推导出"哪段记忆导致了错误判断"。类比能借来组织形态，借不来可推导性。把它当架构约束用是对的，当论证用就是过度延伸——论文自己也只声称借的是"组织程序的方式"，这条自我限定值得后来的引用者遵守。

### Claim-to-evidence：把信任变成可计量的开销

claim 分级（supported、partially supported、prerequisite-supported、evidence-gated）加上 evidence packet，本质上只做一件事：把"我做过"这种声明降级为"这里有可复核的产物"，并把不可复核的部分显式标成 scope limitation。这是信任原语，不是文档规范。

代价很实在。一个 evidence packet 要背命令、manifest、源码与环境元数据、Session JSONL、生成产物，外加"证明什么、不证明什么"的说明——等于每个结论都附一份可复现现场。对研究型工作这是底线；对产品型工作，这会把产出成本抬高一个量级。真正的工程判断不是"要不要 evidence"，而是"哪一层结论需要哪一级 evidence"：对外承诺要 supported，内部探索停在 evidence-gated 就够。把分级当成统一的强制门槛，团队会绕过去；把它当成成本开关，它才活得下去。这一点和 [[concepts/harness-engineering-framework|Harness Engineering]] 里"证据驱动而非断言驱动"的取向是同构的。

### 工具执行边界在哪泄漏

OpenRath 的收口很干净：无论成功、异常、未知工具还是参数错误，最终都落成 Session 里的 tool-result chunk。这一条把"模型声称"改写成"运行时记录"，是全文最可复用的设计。

但边界仍在三处泄漏。其一，schema 与意图之间存在语义鸿沟：参数校验能保证类型合法，不能保证意图合法，参数齐全的 `rm -rf` 照样通过校验。其二，沙箱是执行位置而非权限边界——merge 时检查"两个分支的执行环境是不是同一个意义上的环境"，恰恰说明环境兼容性是人工判定而非形式化的，这也解释了为什么 [[concepts/agent-sandbox|Agent Sandbox]] 这类隔离方案无法直接替代状态层的判断。其三，间接 prompt injection 让"工具读到的东西"变成新的输入源，而 Session 只记录工具返回了什么，不区分这段返回是数据还是指令。论文把安全属性列为未证明项是诚实的，但也说明当前的工具边界是"记录完备、约束不足"。

### 坐标系：它占的是哪个格子

和 [[entities/langgraph-state-machine|LangGraph 状态机]] 比，差别不在能力而在所有权：图状态服务持久执行、resume 和 time travel，读者是调度器；Session 服务 Agent 程序自身，读者是下一段代码。和 [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]]、[[entities/opensandbox-aliyun-cloud-agent-sandbox-vibecoder|OpenSandbox]] 这类沙箱方案比，OpenRath 关心的是沙箱句柄如何随 Session 流动、merge 时如何判定兼容，而不是如何把隔离做得更硬。和 MCP 比，MCP 划的是工具接入的协议边界，OpenRath 划的是效果回到程序后的状态边界，两者正交、可以叠加。

也就是说，它不与这些框架争夺"怎么编排"，而是追问"编排出来的状态归谁"。这也解释了它的局限为什么集中在验证侧：一个关于所有权的主张，缺的从来不是设计，而是 benchmark 与对抗性安全证据。

## 实践启示

1. **先定状态所有者，再选编排框架。** 当系统开始分角色、分工具、分沙箱、分记忆、分支路时，"再加一个 Agent"是最差的默认反应。先回答：这次运行的真实状态由哪个值拥有？答不出来，换框架也治不好——同时也要看清 OpenRath 自身尚未跨过 benchmark、记忆质量与安全属性三关，现阶段它是状态所有权的参考实现，不是可直接托付的生产底座。
2. **把证据挂在传递的值上，而不是日志里。** 事后拼接的 trace 只能支持观测，支持不了"程序继续往下跑时仍能拿到统一状态"。让 Session 作为函数参数流动，而不是作为数据库主键被查询。
3. **用"不拥有什么"划分对象。** 照抄那份负面清单：Agent 不持会话图、Tool 不持执行位置、Workflow 不造私有调度状态、Memory 的 recall/commit 必须进入可见记录。负面清单比正面职责更能防止状态重新散落，因为散落几乎总是从"顺手在这里缓存一份"开始的。
4. **fork/merge 必须带环境兼容性判定。** 分支能不能合，一半是内容问题，一半是"两个沙箱是不是同一意义上的环境"。把这条写进 merge 的前置条件，否则合并出的状态会在下游以极难定位的方式失败。
5. **给结论分级，并按级配证据成本。** 对外结论要 supported 加完整 evidence packet（命令、manifest、环境元数据、JSONL、产物、证明范围）；内部探索停在 evidence-gated。统一强制最高等级，等于训练团队绕过流程。
6. **把工具结果当不可信输入处理。** 记录完备不等于约束完备：参数校验挡不住意图错误，沙箱位置不等于权限边界，工具返回值既可能是数据也可能是指令。这三点要补策略层，不能指望 Session 记录兜底。

## 相关实体

- [[entities/langchain-harrison-chase-sandbox-architecture|LangChain Sandbox Architecture]] — 另一种 Agent 运行时设计
- [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]] — 类似的 sandbox 架构
- [[concepts/harness-engineering-framework|Harness Engineering]] — Agent 运行时的更广泛框架
