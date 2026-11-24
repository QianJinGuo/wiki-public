---
title: "蚂蚁清华开源 Realtime-Venus：9B 全双工交互模型 + Harness 打通前后台"
created: 2026-09-30
updated: 2026-11-17
type: entity
tags: [agent, harness, full-duplex, multimodal, open-source, realtime]
sources: [raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026]
confidence: 0.7
---

# 蚂蚁清华开源 Realtime-Venus：9B 全双工交互模型 + Harness 打通前后台

> **Background**：本文基于量子位对蚂蚁 AI 安全实验室与清华大学人机交互实验室联合发布 Realtime-Venus 的报道整理。核心信息：开源 9B 全双工交互模型（Omni/Audio 两款）+ 配套 Harness，实现"边聊边办事"的异步任务委托架构，对标 TML-Interaction-Small / SeedRealtime / Qwen-Omni-Realtime 等闭源方案。→ [[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026|原文存档]]

## 深度分析

Realtime-Venus 解决的核心问题是实时交互的"阻塞"：查资料、跑任务需要时间，用户却在此期间继续追问或补充信息。系统将主动感知和对话控制交给前端模型，通过配套 Harness 连接后台服务，让任务执行与对话同时进行——用户可随时插话、追问，后台结果返回后模型结合对话状态选择播报时机 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**架构三层能力**：(1) 持续感知——模型持续接收画面和声音，在关键事件前保持倾听并主动提醒（如"裁判吹哨了"），用户交代任务后无需反复追问；(2) 全双工交互——说话时继续接收输入，结合语义判断何时续说、何时让出话语权（"对，没错"是鼓励继续，"等等，我改一下"是要求停下），被纠正时停下回应、调整未说内容再继续；(3) 后台委托——遇外部能力任务委托后台处理，结果带回对话 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**与 Harness Engineering 的关联**：这是少见的将 harness 概念显式应用于语音交互领域的开源实践——前端 9B 模型（感知+对话控制）与后台任务执行通过 harness 解耦，与 [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering]] 中"模型负责认知、harness 负责执行"的分层思路同构。区别在于 Realtime-Venus 的 harness 还承担全双工话语权仲裁（何时打断、何时播报），这是语音模态特有的执行控制问题 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**竞品坐标**：闭源阵营 TML-Interaction-Small、SeedRealtime、Qwen-Omni-Realtime 相继推出全双工方案；Realtime-Venus 以开放权重+代码的姿态入场，支持自主部署、定制与扩展，是全双工交互 + 后台任务委托能力的首个开源组合。Omni 还可借助外部记忆检索历史视听信息（回答与早先画面有关的问题），记忆检索成为长时交互的支撑组件 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

### 全双工的评测口径：续说容易、接打断难

团队用 Full-Duplex-Bench v1.5 把"该不该接话"拆成场景分别测：在用户附和、用户转而对他人说话、背景出现人声这三类应当继续回应的场景里，Audio 的续说率为 97%、88%、86%，均高于参评的 Gemini 3.1 Live 与 GPT-4o；但面对用户真正的打断，响应率只有 75%，落后闭源模型。这组对照说明：判别"不该闭嘴"（过滤附和与无关人声）已可超越闭源阵营，判别"该停下"（识别真实打断）仍是短板。对部署者的含义是：验收全双工助手不能只看平均续说率，打断响应率才是体验瓶颈的直接来源；文本语音栈里用级联 ASR+LLM 做打断检测的方案（参见 [[entities/livekit-agents-voice-ai-streaming-cascade-interruption-detection|LiveKit 级联语音栈的打断检测]]）与这里的语义级话语权仲裁，是同一问题的两种解法。

### Harness 的委托生命周期：上下文快照与任务状态机

原文给出一次委托的完整链路：发起时 Harness 先保存当时可用的对话与视听证据并绑定到任务——用户此后继续补充需求，已发出的任务仍保留发起时的上下文，避免不同时刻的信息混在一起；任务随后交给已注册的后台能力（多模态感知、通用推理或自定义技能）执行；结果返回后整理成适合口头表达的回复送回原会话。系统还记录排队、执行中、已完成、交付中、已送达五种状态，开发者据此能区分"结果未返回""已返回待播报""已播完"。这套"快照冻结 + 状态机 + 时机仲裁"协议与 OpenAI GPT Live 的前后台委托（参见 [[entities/openai-gpt-live-real-time-voice-frontend-backend-delegation|OpenAI GPT Live 前后台委托]]）同构，但把"结果何时开口播报"显式纳入协议——这是语音模态特有的执行控制增量。

### 长视频记忆：无参更新的检索式补偿

通话持续进行时，早期画面会移出上下文窗口，追问"刚才是谁把盘子递给厨师"就可能无据可答。Omni 的长视频记忆模块不走训练路线：按视觉变化等信号筛选历史画面，保留与视频时间线对齐的音频，收到问题后结合语义相关性与画面新颖度检索证据，连同邻近音频和近期视听内容重组为回答上下文。量化收益落在长视频问答基准：LVOmniBench 60–90 分钟子集准确率 41.18%→47.06%，LongVideoBench 40–60 分钟子集 57.14%→61.90%。约 5 个百分点证明检索补偿有效但远未饱和；其"流式窗口当工作集、记忆模块补长期侧"的分工，正对应 [[concepts/working-set-vs-long-term-memory|工作集与长期记忆之分]] 的框架。

### 从"能交互"到"会交互"：三段公开承认的差距

史元春在寄语中把差距说得很具体：自然的交互不止于听和说，还包括理解上下文、预判需求、在合适的时机交付结果。原文恰好为这三点各留了一个待解项——理解上下文对应更长连续交互中如何保留有用信息；预判需求对应更细粒度的流式交互与更长的上下文窗口；合适时机交付对应复杂任务的执行与失败后恢复。团队的路线（继续探索流式交互、长窗口、任务执行与恢复）与开源策略（开发者接入自己的业务工具，在实际使用中检验对话节奏和任务完成情况）表明，官方把这组差距定位为待验证的工程问题而非架构缺陷；评估该项目时，判断标准应是这些公开短板的收敛速度，而非当前榜单名次。

## 实践启示

1. **验收全双工助手要拆开"续说"与"打断"两套指标**：复用 Full-Duplex-Bench 的场景划分（附和/对他人说话/背景人声 vs 真打断）作验收口径，别让平均续说率掩盖打断响应短板（Audio 后者仅 75%）；评测方法论可参考 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准体系]]。
2. **后台委托必须做上下文快照**：任务发出时冻结当时的对话与视听证据并绑定到任务，用户后续补充只进新指令、不回写已发出任务——这是防止多轮异步任务串上下文的最低成本设计。
3. **异步结果需要显式的播报仲裁**：结果返回不等于立即播报；维护排队/执行/交付的任务状态，让前端结合"用户此刻是否在说话"决定开口时机，否则异步助手必然抢话。
4. **长时流式交互优先考虑检索补偿而非扩窗口**：对移出窗口的历史视听信息，按视觉变化筛选、按语义相关性加画面新颖度检索、连带邻近音频重组上下文——无需训练即可拿到数个点的长视频问答收益。
5. **语音 Agent 落地先看 Pass@1 再看 F1**：工具选对（Omni 工具选择 F1 86.0%）不等于事能办成（完整任务成功率 Pass@1 仅 43.0%/42.0%），参数填写与多步执行的失败率才是工程化门槛；集成前要求厂商同时公布两级数字。
6. **小模型 + 模态分工是开源可复刻的现实路径**：两款 9B 模型（Omni 视听、Audio 纯音频）基于 MiniCPM-o 4.5 独立训练、后训练语料超 280 万条，证明全双工交互能力在消费级参数量即可达标；算力受限团队做双专模型比单追大而全更划算。

## 相关

- [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[concepts/openai-realtime-voice-architecture|OpenAI Realtime 语音架构]]

→ [[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026|原文存档]]
