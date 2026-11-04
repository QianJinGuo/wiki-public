---
title: "AI 智能建场：用 ReAct 决策 + 状态机门控，把一句话变成可投放会场"
created: 2026-11-04
updated: 2026-11-04
type: entity
tags: [agent-harness, react, state-machine, tool-runtime, ag-ui, sse, async-callback, metaq, fence, cas, idempotency, production-plan, field-coordinate, tmall, taobao]
sources:
  - raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04
confidence: 0.9
provenance_state: extracted
review_value: 9
review_confidence: 9
---

# AI 智能建场：用 ReAct 决策 + 状态机门控，把一句话变成可投放会场

## 一句话结论

天猫国际的"智能建场"把运营的一句"帮我搭一个七夕轻奢箱包会场"变成可投放页面。它的立场很明确：**不是让大模型生成 JSON，而是建一套 Agent Harness——模型负责理解目标、选工具、根据 Observation 继续决策；状态机、生产计划、字段坐标与任务账本负责限制"能做什么、结果写到哪里、失败后怎么恢复"**。一句话总结全文：**模型决定能力上限，Harness 守住生产可信下限。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 一、三类新问题：为什么"生成页面"不够

自然语言不是生产参数（"七夕轻奢箱包"没有价格、品牌、商品分层、坑位数量——**模型自动补齐=替用户做不可见决策**，逐项追问=AI 退化成表单）；模型决策不等于业务事实（模型说"圈品已完成"、工具返回 `success=true`，但 U+ 可能只是**受理**，真正的 `datasetId` 要等异步回调；投放结果要等用户点"我要投放"）；一次连接覆盖不了完整生命周期（圈品可能持续数分钟，期间用户刷新、断开 SSE 或改同一字段，系统**必须脱离聊天上下文保存任务状态，并判断迟到回调是否仍然有效**）。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

于是目标被重写成一句可验收的话：**把不完整的用户目标转化为可确认的生产计划，让每次工具调用都有明确边界，让跨请求结果能够恢复，并让最终页面的每个字段都可解释。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 二、骨架：ReAct 决定做什么，状态机决定允许做什么

主链路：**输入 → 恢复 Session/Task → 模板缺口审计与澄清 → 用户确认终值或 AI 生成指令 → 字段级 ProductionPlan → ReAct 选工具 → 同步结果写页面、异步结果登账本 → 前端预览与局部修改 → 只重生产受影响字段 → 确认完成**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**两条不能混淆的链路**（全文的枢纽判断）：**推理链路**=模型根据目标、任务状态与 Observation 决定下一步；**事实链路**=Task 保存页面终值、待生成字段与异步终态，**决定系统实际完成了什么**。作者一句话点透：**"对话是入口，Task 才是业务事实。"**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**为什么用 ReAct 而不是固定 Workflow**：会场生产的执行顺序取决于模板与用户选择（有的模板只需生图，有的要生图+生文+圈品；用户可能预览后只改一个字段），**若为所有组合预编排节点，流程会随模板与场景膨胀**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**分工一句话**：**ReAct 决定下一步做什么；状态机决定当前阶段允许做什么。** 状态映射动态工具白名单：^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

| 迁移条件 | 目标状态 |
|---|---|
| `INIT` + `CLARIFY_COMPLETE` | `RUNNING` |
| `RUNNING` + `MATERIAL_GENERATED` | `WAITING_CONFIRM` |
| `WAITING_CONFIRM` + `MODIFICATION_CONFIRMED` | `RUNNING` |
| `WAITING_CONFIRM` + `PUBLISH_CONFIRMED` | `COMPLETED` |

> **状态门控是运行时权限，不是写在 Prompt 里的建议。** 即使模型判断漂移，也无法在需求未确认时提前圈品，或在任务完成后继续写入。

**模型与代码的边界（可直接当分工模板）**：^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

| 模型承担 | 代码必须承担 |
|---|---|
| 理解自然语言目标 | 模板必填缺口审计 |
| 生成摘要与预填建议 | 任务状态迁移与工具权限 |
| 选择当前需调用的工具 | 字段坐标、下标与 Schema 校验 |
| 把自然语言修改翻成候选原子操作 | 用户确认值落盘 |
| 根据 Observation 解释进展 | 工具结果写入与生产账本维护 |
| — | 并发锁、CAS、fence 与异步恢复 |

## 三、Harness 六件套：把模型装进可生产的外壳

**① 请求、身份与流式生命周期**：`AbstractStreamingChatExecutor` 统一完成"**解析登录身份 → 创建或恢复 Session → 获取 Session Lock → 创建或加载 Task → 恢复消息与业务事实 → 执行 Agent → 持久化结果并释放资源**"；`BaseStreamingExecutor` 负责 SSE（**900 秒连接超时、15 秒心跳**、断连/超时/异常后取消 Flux、并行事件发送锁、**Reactor 线程切换后恢复 Trace 上下文**）。生产请求从登录上下文解析 `operatorId`，Session/Task 查询继续携带 `operatorId` 校验归属——**作者明确这是"身份解析与数据归属隔离，而不是完整 RBAC"**（这种自我限定的写法很值得学）。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**② AG-UI 事件基座**：通用协议覆盖 `RUN_STARTED/FINISHED/ERROR`、`TEXT_MESSAGE_*`、`REASONING_*`、`TOOL_CALL_START/END`、`CUSTOM`；**场景只定义自己的 `CUSTOM` 卡片**（建场：`clarification`、`modificationProposal`、`venueResult`；AI 设计可扩展设计方案卡、素材选择卡、画布结果卡）。目的很硬：**前端不根据"我已经完成"这类模型文本猜状态，而根据工具事件、Task 结果与业务卡片驱动交互**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**③ Session / Task / 状态机**：四类持久化对象承接完整生命周期；**对话上下文可以重新构造，但 Task 中的业务事实不能依赖模型记忆**；状态机在这里**不是流程展示，而是运行权限系统**（`TaskStatus` → 动态工具白名单、`ProductionPlan` → 可操作字段、**CAS 保证旧请求不能覆盖新阶段**）。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**④ Tool Runtime**：`AbstractStreamingAgentGateway` + `StreamingToolCallHook` 共同构成——**先按状态过滤工具**并包装 ToolCallback，再做**坐标预检、Run 级原子防重、TOOL_CALL_START/END、工具计数、dependency.log 与 toolcalllog 留痕**；执行完成后**在 Task 锁内**委托 `ToolResultPostProcessor` 写业务事实，最后做**模型表述与真实轨迹的一致性审计**。原则一句：**"Prompt 约束模型的行为倾向，Tool Runtime 约束真实执行权限。"** 另有三条重试纪律：失败且未产生副作用时可撤销轮内坐标并有限重试；同一工具连续失败达阈值本轮熔断；**只读或幂等工具可在审计命中时安全重放，圈品/生图等生产型副作用工具不能因为模型声称"已完成"而自动补执行**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**⑤ 异步运行时**：**把"一次 Agent Run"与"完整业务任务"解耦**——同步 Run 只登记"外部任务已受理"，`Tair` 保存回调上下文，SSE 可结束；MetaQ 到达后重新加载 Task、在 Task Lock 内修改事实、用 `createBizId` fence 判断结果是否仍代表当前意图。不止圈品，也能承接异步生图、视频生成与批量设计。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**⑥ 运行证据**：`AgentRunLog`（Run、TTFT、Token、工具次数、熔断与审计）、`toolcalllog`（真实工具输入输出）、`dependency.log`（LLM/TOOL/HTTP/HSF/TAIR/METAQ 依赖耗时与错误）、Trace 串起 **HTTP → Reactor → 工具线程 → 回调线程**。故障定位最终要回答六个问题：**参数来自哪里、工具是否真实执行、是否被防重或熔断、是否产生副作用、结果写入哪个坐标、回调为何命中或被丢弃。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 四、工具：业务动作与生产权限，不是底层接口列表

**为什么不直接暴露底层接口**：底层服务围绕系统能力设计（创建圈品任务、查预览商品、调生图接口），而模型需要的是围绕业务目标设计的动作（"为指定模块坑位圈选商品"）。三条原则：**①一个工具表达一个模型可理解的业务意图；②参数只保留完成该意图所需的最小信息；③外部调用编排、协议转换与结果校验封装在工具内部。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

五个核心工具：`analyzeRequirements`（识别模板缺口并生成澄清卡）、`selectProducts`（提交条件给 U+，返回异步受理结果）、`generateImage`、`generateCopy`、`proposeModification`（把自然语言修改翻译为待确认原子操作）。**它们不组成固定流水线，由 ReAct 按 Task 状态、ProductionPlan 与上一轮 Observation 动态选择。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**统一业务坐标**：`moduleUuid + moduleName + slot + key + index`（`moduleName` 定位模块类型、`moduleUuid` 区分同名实例、`slot` 定位 Schema 资源位、`key` 定位字段、`index` 定位数组中的一组）。**模型不是自由编造坐标，而是从 ProductionPlan 中选择**；工具侧再校验坐标是否属于模板与本轮计划，后处理按同一坐标写入。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**一个结果，三类消费者**：ReAct 当 Observation 判断下一步；Task 由后处理器写终值或过程状态；前端通过工具事件与 `venueResult` 展示真实进度。例：`selectProducts` 返回 `createBizId` 后，**模型观察到"已受理"、Task 把坐标置 `GENERATING`、前端显示"正在圈品"**——同一事实，三种呈现。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**按副作用设计错误与重试**：只读/提案工具（可安全重放）/ 同步生产工具（未产生副作用时有限重试）/ 异步副作用工具（**受理后不能因等待或本地异常机械重提**）。错误协议必须表达**发生了什么、是否已产生副作用、能否重试、需要用户改什么**——"统一返回'调用失败'会诱发模型盲目重试"。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 五、需求工程化：从一句话到字段级计划

**先审计模板，再询问用户**：前端首次请求带 `templateId`、`templateSchema`、类型化 `content`；系统**不把整个模板交给模型自由判断缺失项**，而是按模板必填模块、`need_clarify` 字段与当前 `renderResult` 做**确定性裁剪**，LLM 只负责理解表达、生成摘要与预填建议。两个收益：**必填字段不会因模型漏判而消失；模型服务失败时仍可退化为一张空白澄清卡，不让链路中断**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**澄清卡是输入协议，不只是交互组件**：下发内容含需求摘要、缺失字段、已识别字段与预填值，`missingFields` 与模板 Schema 同构（前端无需回查模板）。提交用 **`value + fillMode` 双树**：`manual` 或缺省 → 同坐标值是**终值**直接写 `renderResult`；`ai` → 同坐标值是**生成指令**，登记到 `pendingGenerations`。**用户确认值不再交给外层 LLM 二次解释**——对圈品尤其关键：`tabName` 只负责展示，**用户没填的价格、爆品、商品分层不能被静默加入真实条件**。作者的判据一句到位：**"卡片所见即工具所执行。"**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**ProductionPlan 字段级**：每项含字段坐标、`operationType`（image/text/product）、`imageSize`（从 `x-format` 解析）、`referenceImageUrls`（`x-reference`）、`description`（用户确认的生成指令）、`aiDescription`（模板固有生产约束）、`optional`。**`description` 与 `aiDescription` 刻意分离**：前者是"用户这一轮想要什么"，后者是"模板长期要求的尺寸、字数、风格边界"——**用户填写新指令不能覆盖模板约束**。账本规则同样严：有 PENDING 只做 PENDING；**账本整体为空才允许按 Schema 构造全量计划**；**只有 `GENERATING` 或 `FAILED` 时禁止回退全量扫描**，否则会重复生成正在执行或已失败的字段。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 六、异步：同步只受理、回调续跑、fence 丢弃迟到结果

**为什么不能原地等**：U+ 圈品可达分钟级——在线程内轮询会长期占用连接，超时后留下"不知道外部是否已执行"的不确定状态；**模型若因此自动重试，还可能创建多个真实圈品任务**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

- **第一段（同步受理）**：调用 `selectProducts` → 标准化条件 → 拿到 `createBizId` → 上下文写 Tair → 账本坐标置 `GENERATING` → **本轮 SSE 正常结束**（回调不依赖原模型上下文或连接）；
- **第二段（MetaQ 回调续跑）**：按 `createBizId` 读上下文 → 取 Task 级锁 → 在 `pendingGenerations` 查受理号 → 回填业务终值 → 销账 → 写 `selectionRecords` → 保存并释放锁。**回调侧只更新业务事实，不直接驱动状态机**，阶段仍由正常对话编排推进；
- **迟到回调**：用户等待期间改了同一字段 → 新操作 `resetToPending`、清空旧 `createBizId`——**旧回调无法在账本命中受理号，判定为已被取代并丢弃**。作者命名为"**面向业务坐标的 fence**：不要求外部回调严格有序，只要求结果写入前仍然代表当前用户意图"；
- **前端恢复**：`moduleStates=GENERATING` 时整模块显示占位并按 **10–30 秒**轮询，清空后停止；**刷新页面也能从 Task 恢复，连接不再是任务的事实来源**。

三个必须分清的概念（这段是全篇的"金句区"）：**工具调用成功 ≠ 圈品业务完成 ≠ 页面终值已写入**；`RUN_FINISHED` 只表示本轮 Agent 调用结束，**不能因此停止仍在执行的圈品轮询**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 七、人机协同：局部修改，不重生成整页

`WAITING_CONFIRM` 状态下，模型用 `proposeModification` 把自然语言翻成三类原子操作：**`APPEND_ITEM`（向数组资源位追加）/ `UPDATE_FIELD`（更新终值或登记新的生成指令）/ `REMOVE_ITEM`（删除指定数组项）**；提案携带 `moduleUuid`、字段坐标、当前值、建议值、字段标题与 Schema 定义，后端先校验模块实例、数组下标与操作冲突，再通过 `modificationProposal` 卡片交用户确认。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**确认后的操作不再由模型解释**：前端回传用户确认或微调后的 `operations`，应用层按**确定性顺序 `APPEND_ITEM → UPDATE_FIELD → REMOVE_ITEM`** 执行；`manual` 直写 `renderResult`，`ai` 把账本项重置为 `PENDING`；**只为受影响坐标生成新的 ProductionPlan，已确认的其他商品、图片、文案保持不变**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**不确定时不猜测**：无法唯一定位模块、下标越界、同一资源位互斥修改时，把原因放进 `rejected` 要求用户确认目标——**"生产系统宁可多一次确认，也不让模型猜测修改真实页面"**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 八、可靠性治理：防重键是业务坐标，不是工具名

**重复调用**：同一轮可能合法地为多个模块调 `generateImage`，所以防重不能只看工具名——真业务键是 `toolName + moduleUuid + moduleName + slot + key + index`，以 **Run 级 Concurrent Set 原子 check-and-add**，由 `StreamingToolCallHook` 统一预检。三层治理：**执行层**（同坐标在途或已成功则不再产生副作用）、**观测层**（重复决策仍写 `tool_call_log`，记参数与拦截原因）、**交互层**（没有真实执行的拦截，**不显示成"已执行工具"卡**）。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**三类并发，五种机制，不可互相替代**：^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

| 机制 | 解决什么 |
|---|---|
| **Session 分布式锁** | 同一 `sessionId` 只允许一轮对话，冲突返回 `SESSION_BUSY` |
| **Task 级锁** | 串行化工具后处理与异步回调对同一 `resultJson` 的读改写 |
| **CAS 状态迁移** | 只有旧状态符合预期才推进，防旧请求覆盖新阶段 |
| **SSE 发送锁** | 防并行工具事件破坏帧顺序 |
| **`createBizId` fence** | 阻止过期回调覆盖用户新操作 |

> "**锁解决并发执行，CAS 解决状态迁移，fence 解决跨时间结果有效性。**"

**一致性审计**（最有意思的一段）：模型可能在 `WAITING_CONFIRM` 声称"已经生成修改确认卡"，而本轮**实际没调用** `proposeModification`。最新实现把审计收敛到 `AbstractStreamingAgentGateway`，子类只声明判定策略；命中"**宣称已执行但轨迹缺失**"时，框架通过包装后的 ToolCallback **自动重放** `proposeModification`，使 TOOL_CALL 事件、日志、结果捕获与卡片分发仍走原链路；重放失败则补发纠偏文本。边界很清楚：**只读工具可在明确条件下重放，有副作用的生产工具不能因为模型说"已完成"而补执行**——**判断依据永远来自 Tool Trace 和 Task，而不是自然语言**。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

**可观测性串因果链**：`sessionId → taskId → runId → toolCallId`，以及 `└→ createBizId → datasetId → displayId`。消息时间线记"人看到什么"、`tool_call_log` 记"模型实际调了什么"、`dependency.log` 记外部依赖耗时与错误类型、`AgentRunLog` 记一次 Run 的 TTFT/token/工具次数/终态/是否熔断，Trace 连接 HTTP、Reactor、工具线程与回调线程。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 九、评测：页面像不等于过程稳

**Tool Trace 轨迹评测七问**（可直接当验收清单）：^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

1. 是否在正确状态调用了允许的工具？
2. 工具参数是否来自用户确认或可信 Task？
3. 是否添加了用户未提供的圈品条件？
4. 是否对同一坐标重复调用？
5. 异步受理后是否错误重提？
6. 结果是否写入正确字段？
7. 需要人工确认时是否越权继续执行？

作者的判断：**"最终页面相似，不代表执行过程同样可靠。"**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 十、模型与 Harness 的七条总结（原文照录要点）

1. **ReAct 负责动态决策，状态机负责业务边界**——两者结合才同时获得灵活性与可控性；
2. **工具是业务动作与生产权限，不是底层接口列表**——参数、结果、错误都要围绕模型决策设计；
3. **用户确认必须进入确定性通道**——澄清卡与修改卡是**协议边界，不是 UI 装饰**；
4. **Task 才是业务事实**——页面终值、生产过程与异步结果必须结构化持久化；
5. **前端协议属于 Agent 能力**——没有事件、卡片与恢复机制，长流程 Agent 无法形成稳定体验；
6. **通用基建决定场景扩展成本**——流式执行、事件协议、工具框架、会话任务、异步回调与观测可继续支撑 AI 设计等场景；
7. **数据效果观测体系**——投放后的业务使用情况与 A+ 数据要有迹可循。

以及那句总纲：**"模型负责理解和规划，协议负责协作，状态机负责边界，工具负责执行，Task 与事件负责将长流程沉淀为可信的业务事实。"**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 深度分析

### 这篇的真正主题：把"模型的判断"降级为"系统的状态"

全文所有机制都在做同一件事——**不让任何关键事实由模型的语言承诺来定义**：能不能做由状态机决定、写到哪由坐标决定、做完没有由账本决定、迟到结果是否有效由 fence 决定、是否真的执行由 Tool Trace 决定。**模型被降级成一个"提出下一步意图"的组件，而系统用自己的状态定义现实。**这是长流程 Agent 与 demo 的分水岭。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### "对话是入口，Task 才是业务事实"解决的是**事实来源**问题

绝大多数 Agent 崩溃在"上下文丢了"这个层面——但真正的故障是**事实来源错了**：把对话历史当真相，一旦 SSE 断开、用户刷新、上下文被截断，系统就失去"做到了哪一步"。这里的解法是把事实**外置到 Task + 账本**，对话只负责触发与解释。**推论：任何跨会话、跨连接、跨小时的 Agent 都必须先回答"业务事实存在哪"。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### 字段坐标是这套系统的"类型系统"

`moduleUuid + moduleName + slot + key + index` 五段坐标同时承担四件事：**模型的选择范围**（只能从 ProductionPlan 选）、**工具的参数校验**（是否属于模板与本轮计划）、**结果的写入位置**、**防重的业务键**。没有坐标，就必须靠模型复述 JSON——而模型复述 JSON 是这一切不可靠的源头。**把"写哪里"变成结构化主键，是让 Agent 能安全操作复杂对象的通用手法。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### `description` vs `aiDescription` 的分离，是"用户意图"与"系统约束"的正交化

用户说"写一条轻奢感七夕文案"，模板说"8–12 字、不得换行"——两者**共同约束、互不覆盖**。这条设计可以推广到任何"用户输入 + 领域规范"并存的场景（代码生成里的风格约束、医疗记录里的必填项、财务文案里的合规条款）。**大多数"AI 不听话"其实是把两类约束塞进了同一个字段。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### 面向业务坐标的 fence：松散一致性的正确用法

外部回调无法保证有序，也无法保证及时。这里的处理不是"加超时后放弃"，而是**给每次受理一个业务身份（`createBizId`），并在写入前检查它是否仍是当前意图的受理号**——过期即丢弃。这套思路与分布式系统里的 fence token 同源，但**落在业务坐标上而不是存储位置上**，因此不需要外部系统配合改造。**任何"异步结果可能晚到"的场景都能直接套。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### 一致性审计只重放只读工具，是一条安全红线

"模型说做了但没做"是极常见的幻觉形态。这里的处理很聪明：**框架自动重放只读/提案类工具**（补上事件、日志、卡片，让用户看到的东西与事实一致），**但绝不代执行有副作用的工具**。**审计的对象是"表述与轨迹的差异"，而不是"结果好不好"**——边界划在这里，才不会把审计变成第二个执行者。^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

### 自我限定的写法值得学

两处都写得很克制：身份部分**"属于身份解析与数据归属隔离，而不是完整 RBAC"**；工具结果不夸大、评测只给七问不给漂亮总分。**在工程文章里，明确"没做什么"比声称"全做了"更有信息量。**^[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04.md]

## 实践启示

1. **先回答"业务事实存在哪"**：对话是入口，Task/账本是事实；否则 SSE 一断就失忆。

2. **状态机当权限系统用**：状态 → 工具白名单，模型漂移也无法越阶段执行；门控写在运行时，不写在 Prompt。

3. **字段级坐标（module/slot/key/index）**：让"写哪里"成为主键，同时服务选择范围、参数校验、结果写入与防重键。

4. **按副作用给工具分类**：只读可重放、同步可有限重试、异步副作用**受理后禁止机械重提**；错误返回必须说明"有无副作用/能否重试/用户要改什么"。

5. **澄清卡做成协议**：`value + fillMode` 双树区分"终值"与"生成指令"，**用户确认值不再交给 LLM 二次解释**——卡片所见即工具所执行。

6. **用户意图与系统约束正交**：`description`（本轮想要）与 `aiDescription`（模板长期约束）分字段传，互不覆盖。

7. **账本三分态（PENDING/GENERATING/FAILED）**：只做 PENDING；整体为空才全量；**有在途或失败时禁止回退全量扫描**。

8. **异步用两段式**：同步只登记受理、上下文入缓存；回调在 Task 锁内改事实；**用业务受理号做 fence 丢弃迟到结果**。

9. **锁/CAS/fence 三件套不要互相替代**：并发执行、状态迁移、跨时结果有效性各管一段。

10. **一致性审计只重放只读工具**，判断依据永远是 Tool Trace 与 Task，不是模型的话。

11. **评测要评轨迹**：七个问题（正确状态/参数来源/有无私自加条件/重复调用/错误重提/写入坐标/越权执行），**页面像 ≠ 过程稳**。

12. **前端协议是 Agent 能力的一部分**：三类信息分开（模型说明/工具事件/业务卡片），`RUN_FINISHED` 不等于异步任务结束。

## 与相关实体的关系

- [[entities/deepseek-super-agent-design-router-worker-architecture-2026-10-31|DeepSeek Super-agent]] 给的是"连续/高效/主动/持久"的骨架（双层、事件驱动、存算分离、Watchdog）；本篇是**这套骨架在一个真实业务里落到字段级坐标、账本与 fence 的完整实现**，两者对照读最有价值。
- [[entities/coworkos-agent-coworker-ruc-fudan-2026-11-03|CoworkOS]] 也有"持久工作状态 + 后台 Job + 版本核对/候选合入"；区别是本篇面向**结构化页面资源的字段级写入**，CoworkOS 面向**文件/BIM 类作品的版本协作**。
- [[entities/taobao-d2c-harness-three-platform-visual-verification-2026-11-02|淘宝 D2C 视觉验证闭环]] 同属淘天系 harness 实践，那篇管"生成结果对不对"（四层视觉门禁），本篇管"生成过程可控不可控"（状态机/坐标/账本）——**一个评结果，一个管过程**。
- [[entities/tencent-enterprise-agent-workbuddy-boundaries-infoq-2026-11-03|腾讯企业 Agent 边界]] 的"隐性错误率 + 执行轨迹回放 + 权限重设计"与本篇的 Tool Runtime 治理、轨迹评测是同一问题的两种答法（企业平台 vs 单一业务深水区）。
- [[entities/skill-routing-description-self-evolution-qwen-platform-2026-11-03|千问 Skill 路由]] 的"三类证据不可互替（能力/职责/轨迹）"与本篇"判断依据来自 Tool Trace 与 Task 而非自然语言"是同一条纪律的两次表述。

→ [[raw/articles/tmall-intl-ai-venue-building-agent-harness-taobao-2026-11-04|原文存档]]
