---
title: "Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆"
created: 2026-08-15
updated: 2026-09-27
type: entity
tags: [ai, agent, harness, hook, governance, hitl, 数据工程, 腾讯]
sources: [raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆

腾讯 DECO（跑在生产上的数仓 Agent 引擎）实践系列之护栏层：用 Agent 框架的 Hook 切面，把 LLM 处理长文本时的「偷懒」（截断、略写、残缺）、对生产环境的「越权」（未确认发布、回刷）以及上下文传递中的「失忆」（改了表不查风险、产出了物不知汇报），在代码层确定性兜底——prompt 管不住的，框架来堵。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

核心判断是：这三类问题不是模型能力不够，而是「图省事」或「自作主张」的结构性倾向。长 SQL 物理上超出 token 预算，危险操作是模型无法区分「查询」与「发布」的可逆性差异，被动探测是模型追求最短完成路径的自然倾向——唯一解法是在 Agent 框架层，让偷懒和越权的路径代码级强制走不通，让失忆的已知盲区确定性补齐。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

## Hook 链：在关键切面挂载护栏逻辑

拦截切入口是 Agent 框架普遍提供的 Hook（Callback）机制：框架在「模型调用」和「工具调用」的执行前/后暴露切面，拦截逻辑挂载到切面上，到点自动回调，同一切面可挂多个、按序执行。核心切面包括 Before Tool（工具真正执行前，可改入参、可直接拦截）、After Tool（工具执行后、结果回给 LLM 前，可改返回值）、Before/After Model 与 Before/After Agent。设计原则是基础设施和推理逻辑解耦——Hook 切面上的逻辑独立运作，模型的 ReAct 循环不用感知。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

## 长文本完整性护栏：读写两侧 offload + 引用句柄

数仓长 SQL 的偷懒是结构性问题，解法是「把 LLM 必须接触的长内容降到最少、每次接触的窗口压到最小、所有写入路径都做成小步增量改 + 强制校验」。具体做法是 LLM 永远不直接接触脚本全文：长内容全文留在沙箱，上下文里只有一句引用句柄；拉取侧 Offload Hook（afterTool）把 `scriptContent` 全文写入沙箱只读快照、响应替换为引用句柄，写回侧 Onload Hook（beforeTool）从沙箱读全文覆盖入参、剥离 `scriptFilePath` 字段。落地效果是修改任务工具调用输出 token 直降约 90%，「view → 重写」复印路径下近 100% 的自截断概率被物理消除。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

失败语义按代价差异化：读侧 Offload 落盘失败则降级透传原内容（承担自截断风险、不阻断主流程），写侧 Onload 文件不存在则抛异常阻断工具调用（杜绝发布残缺脚本）。这与 [[concepts/agent-security-architecture|Agent 安全架构]] 中「护栏必须在框架层而非 prompt 层」的思路一脉相承。行业对比上，ADK 与 LangGraph 都只有读侧 offload，DECO 的数仓场景需要两端对称 offload 并额外加固写侧。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

## 危险操作确认（HITL）：beforeTool 卡住不可逆操作

prompt 是软约束，不是安全边界。任何「做了就回不去」的操作（发布、回刷、冻结/解冻、终止）都必须有代码级强制确认：HITL 本质是一个特殊的 beforeTool Hook——工具真正执行前判断「是不是危险操作、用户授权了没」，没授权就阻断。危险工具清单是配置驱动的（yaml），每个配一个授权标记（`requiredState` key）和确认对话框，支持带输入控件的选项（填审批人、填回刷日期），不只是 yes/no。确认动作只能由真实用户在前端触发，Agent 无论自作主张还是被诱导，`packCommit`/`deployCommit` 在框架层都物理走不通。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

## 上下文联动闭环：Hook 采集 → state → Attachment 注入

「主动探测 = 额外一次 tool call = 多耗 token」，模型不会主动给自己加检查步骤，因此要把「副作用采集」和「上下文注入」解耦成两段：采集是确定性的（工具调用一定触发 Hook，不靠 LLM 记得去查），注入是时机正确的（结果只在下一轮 prompt 注入，不污染当前轮）。RiskAnalysisHook 挂在 afterTool 上，通过「带 `tableId` 参数的 upsertTable 才是改表」的精确判定触发下游风险分析；PythonImageHook 通过 before/afterTool 前后文件快照对比发现脚本新产出的图片，生成预签名 URL 注入前端渲染。这套机制把「LLM 需要主动查」降维为「框架主动 push」。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

同一套 Hook 链上 DECO 实际挂了十余个 Hook，覆盖长文本护栏、危险操作护栏、工具返回处理、可观测与持久化、前端实时刷新、Hook→Attachment 联动、沙箱环境等横切关注点。一句话总结：prompt 定意图，Skill 定规矩，框架 Hook 定边界——能用确定性兜底的，别交给模型。^[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆.md]

## 深度分析

把原文拆开看，三类问题其实对应三种不同性质的结构性缺陷，因此治理手段的形态也完全不同。偷懒的本质是**物理约束**：上千行的长 SQL 无法在单轮输出预算内完整重写，「view → 重写」路径下自截断概率接近 100% 不是模型偶尔犯错，而是这条路径必然失败——所以解法是把长内容从上下文里物理移走（读写两侧 offload + 引用句柄），让失败路径根本不存在。越权的本质是**语义盲区**：模型把发布和查询视为同一类「完成任务的步骤」，无法自行区分可逆与不可逆——所以解法是在 beforeTool 切面做代码级门禁，把「可逆性判断」从模型手里收回。失忆的本质则是**信息不对称**叠加**最短路径倾向**：LLM 只能通过工具返回值了解世界，返回里没提「生成了 chart.png」它就根本不知道有东西该查，而主动探测又意味着额外一次 tool call 的 token 开销——所以解法是把「采集→注入」做成框架侧的确定性闭环，不依赖模型自觉。

一个容易被忽略的深层设计是**失败语义的非对称性**。读侧 Offload 落盘失败选择降级透传（承担自截断风险），写侧 Onload 文件不存在却选择抛异常阻断（杜绝发布残缺脚本）。这不是实现上的随意，而是按「出错的代价」做的差异化决策：读失败最多退回治理前的状态，写失败则会把残缺脚本直接送进生产。对称的 Hook 架构配上非对称的失败策略，说明这套护栏的设计单位是「风险」而非「功能」。同样体现风险思维的还有 `scriptFilePath` 框架协议——下游对到达时仍存在的该字段留 `log.warn`，把它当作 Hook 失效的防御性信号。

从行业坐标系看，DECO 的自研边界划得很清楚。长文本护栏上，ADK `ArtifactService` 与 LangGraph DeepAgents 的 offload 都只做到读侧，而数仓场景的长产物要原样回写平台，必须两端对称并额外加固（只读快照/工作副本分离、按字段名识别注释块、列级 offload）——这是「流水线问题」与「单点功能」的差异。HITL 上结论恰好相反：ADK `ToolConfirmation` 与 LangGraph `interruptOn` 的通用拦截已经很好，DECO 自研的必要性「不在于框架没有 HITL，而在于框架的 HITL 不够业务化」——变更预览、带参数确认框、配置驱动的危险清单、与 SSE 流式协议一体，这四条业务级集成需求通用拦截不覆盖。上下文联动上，各框架的 state/checkpointer 都是「存储→读取」的被动模式，DECO 的 Hook→state→Attachment 是「事件→采集→注入」的主动流水线，即使模型完全不知道 state 里有结论也会被「喂到嘴边」。这套横向对比的判断标准值得沉淀：**框架原生能力覆盖的是通用形态，业务形态一旦偏离通用形态，自研点就出现在偏离处**。

另见 [[concepts/context-window-economics|上下文窗口经济学]]——offload 把修改任务的工具调用输出 token 直降约 90%，本质是上下文预算的重新分配：长内容从对话历史和 token 消耗里「隐身」，预算留给推理而非复读。与 [[entities/xiaomi-harness-engineering-prompt-to-hook-to-plugin|小米 Prompt→Hook→Plugin 演进]] 的路径对照，也能看到「护栏从 prompt 层下沉到框架层」是多家团队的共同收敛方向。

## 实践启示

- **prompt 管不住的问题分三类，先分类再选武器**：物理超限（长文本）、语义盲区（可逆性）、激励错位（最短路径）。前两类要框架层强制，第三类要确定性补齐——在 prompt 里加 ⚠️ 对三类都无效。
- **让失败路径物理走不通，而不是靠模型「记得」**：长内容全程走沙箱文件通道、上下文只留引用句柄、写入强制 `str_replace` 小步改；「复印式重写」这条必然失败的路径被移除后，问题从「降低概率」变成「消除概率」。
- **危险操作的守卫必须配置驱动且落在 beforeTool 切面**：危险工具清单放 yaml（不同租户/环境可不同）、每个配 `requiredState` 授权标记、确认动作只能由真实用户在前端触发——Agent 无论自作主张还是被诱导都物理调不通。
- **失败语义按代价差异化**：读侧降级不阻断主流程，写侧阻断杜绝脏数据进生产。护栏设计的单位是风险等级，不是功能对称。
- **「该做没做」类问题别指望模型自查，用 Hook→state→Attachment 闭环 push**：采集是确定性的（工具调用必触发 Hook），注入是时机正确的（下一轮才注入，不污染当前轮）；把「LLM 需要主动查」降维为「框架主动 push」。触发条件要做精确判定（带 `tableId` 的 upsertTable 才算改表），避免误触发。
- **自研前先对照框架原生能力，自研点画在业务偏离处**：yes/no 级 HITL 用原生即可（暂停/恢复、防循环恰是自研最易出 bug 处）；只有变更预览、带参确认、配置驱动清单这类业务级需求才值得自研。offload 同理——ADK/LangGraph 读侧现成，写侧 onload 才是自己要补的。
- **渐进迁移的技巧：互补参数 + 防御性日志**：`scriptContent`/`scriptFilePath` 双参数让下游无感知，Hook 独立演化；下游对「本应被剥离却还在」的字段留 warn，作为 Hook 失效的兜底信号。

## 相关实体

- [[entities/agent-hooks-programmable-workflow|Agent Hooks 可编程工作流]]
- [[entities/deco-agent-hook-governance-tencent-2026|腾讯 DECO Hook 治理]]
- Harness Gate 评估
- [[concepts/agent-memory-architecture|Agent 记忆架构]]

→ [[raw/articles/agent-治理用-hook-堵住-llm-的偷懒越权与失忆|原文存档]]
