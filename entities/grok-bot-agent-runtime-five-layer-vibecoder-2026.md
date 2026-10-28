---
title: "Grok Bot 0.18 运行时重建：Agent 的五层运行时与可靠性协议"
created: 2026-08-26
updated: 2026-10-25
type: entity
tags: [agent, runtime, harness, anysphere, grok-bot, architecture, reliability, context, approval, reply-delivery]
sources: [raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Grok Bot 0.18 运行时重建：Agent 的五层运行时与可靠性协议

> VibeCoder 对 Grok Bot 0.18.0 非官方重建仓库的源码级解读：145 万行运行时 CJS bundle 被恢复成可读 TypeScript，逐层拆解 Anysphere 的 Agent 运行时，提炼出五层架构与五条可靠性协议。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

## 背景：这是 Grok Bot，不是 Grok Build
这次重建对象是 Grok Bot——8 月 11 日进入 early beta、拥有持久云电脑、能跨浏览器/文件/终端/插件工作的 AI 队友；与 7 月官方开源的终端编码 Agent Grok Build 是两个产品。固定样本由 Anysphere Developer ID 签名并通过 Apple 公证，无标准 source map、无官方 monorepo，作者定性为"高保真实现重建"，接近"实现泄露"但离"原始源码仓库泄露"还有证据距离。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

## 五层（六层）运行时架构
顺着一次 sendPrompt 读下去，可见六层：Renderer、Electron main/preload、Node coordinator、Host/transcript、Agent runtime、末端执行 surface。Renderer 管消息/卡片/Computer 画面；preload 只暴露窄 bridge 与单 owner 的 MessagePort；Coordinator 接住 renderer RPC 再连接长期运行的 Host；Host 持有 transcript、幂等接收、每 Agent 的 turn lane 与用户可见状态。进入 turn shell 后系统才建立 generation、权限 epoch、隐私模式、MCP discovery 与 box readiness；Production owner 把本轮 resource accessor、prompt session、summary session 与 tool host 绑好后 runStream 恢复 conversation state 进入模型与工具循环。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

两个稳定细节：输入端用 nonce + digest 做 durable acceptance（网络重试不产生重复回合）；状态端用 generation fence（旧 turn 被新消息打断后，即使晚到也不能覆盖新状态）。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

## 可靠性五协议
**Prompt 是合同，middleware 负责巡检，settlement 负责验收。** 普通 assistant text 只服务内部活动流，只有 SendMessage 才算真正发给用户。start-of-turn middleware 在首次回复前塞入"先用工具"提醒；send-message middleware 在连续工具调用超阈值而无消息时催汇报；turn 结束 transcript runtime 检查 SendMessage 是否真的进入用户可见记录。System prompt 按固定顺序拼接 profile/身份/时区/memory/automation/workflow/channels/Agent directory/MCP/box/Computer 能力，profile 与 memory 在同一 compaction epoch 内冻结。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

**Toolset 是策略。** 每轮重建 toolset：主 Agent/Subagent/共享房间/Computer 未就绪/MCP 未发现拿到不同工具，Shared room 缩成白名单，Subagent 配置缺失直接得空工具集；无 resource accessor 的能力不出现在 schema，执行器缺失 fail closed。低频 MCP 工具用二级发现（先 meta tool，live projection 成立后再加载 connector）减少上下文浪费。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

**Context 要分层。** 四条通道各管一件事：Summary（对话语义，unused context <10k tokens 或 10% 启动后台摘要、5k 或 5% 才允许持久化；输入过大用 max-min fair allocation）、Durable Blocks（计划/待办/项目根/运行模式/自动化/技能）、Memory（profile 与 recent，user/project/agent 不同预算与写入边界）、GUI latest image（视觉状态穿过 compaction）。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

**Approval 要绑定状态。** Local-tool permission（never/ask/always）一次批准绑定 Agent/tool call/动作/目标/direction epoch，新 turn 退休旧批准；Auto-review 判断副作用是否越策略。Computer 操作把动作/目的/box/页面 display-state identity 纳入批准，执行前重查页面，窗口变化旧卡失效；脚本执行补 SHA-256 与规范化 target。安全边界：Electron 开启 context isolation、关闭 Node integration，但 Chromium sandbox 关闭并加 no-sandbox，不能宣传成 OS 级隔离。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

**Reply 是交付。** 把回复当可恢复 side effect：SendMessage 支持文本/图片组/附件/widget/secret/thread；turn result 记录发送数与 reaction，两者都无则视为交付义务未完成，最多三次隐藏 nudge；Ack obligation 持久化，进程中断/重启后可 redrive；无法确认旧动作完成时要求用户重发不假装成功。由此 reply 获得与文件写入相似的可靠性语义：接收幂等、执行可中断、状态防回滚、交付可补偿。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

## Router 属于扩展
Router 支持 Cursor/Claude Code/Codex/OpenRouter + 本地 Docker box，但这是重建作者明确列出的扩展：非 Cursor provider 在 Coordinator 处分流（Claude Code 走临时 MCP bridge、Codex 走 Responses SSE、OpenRouter 走 AI SDK，最多 8 turn/step），没有完整经过原生 Host 的 prompt assembly、memory freeze、durable conversation state 与长循环。它证明同一套 UI 协议可挂多个 provider，不证明原生 Agent 就是简化 Router。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md]

## 深度分析

### 分层依据是失败类，不是功能清单

按功能读，Renderer 是视图、preload 是样板；按失败类读才立得住：每层吸收一类上层处理不了的错误。

- **Renderer / preload** 吸收权限扩散——窄 bridge 加单 owner MessagePort 不是省接口，而是让被污染的渲染进程拿不到执行能力。**Coordinator** 吸收连接抖动——Renderer 会刷新、崩溃、换窗口，Host 必须活得比它们久，接住 RPC 再转给长期 Host，把"进程寿命不一致"挡在状态外。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:22-24]
- **Host / transcript** 吸收三种互不相干的状态错误：nonce 加 digest 治重试造成的重复回合，turn lane 治两回合抢同一份状态，generation fence 治旧回合晚到覆盖新状态。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:22-24]
- **Agent runtime（turn shell）** 吸收跨轮资源泄漏：generation、权限 epoch、MCP discovery、box readiness 都在进入 turn 后才建立，等于给能力标了生死点；本轮 session 与 tool host 一次绑好再 runStream，避免半新半旧。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:22-24]
- **末端执行 surface** 吸收越权副作用：文件、终端、GUI 各有独立审批面，批准谁不连带谁。

于是层数不是分类学问题，而是失败类清单的长度：某层说不清自己吸收哪类错误，它多半只是功能分组。Router 旁路正好反面验证——它绕过的不是某段代码，而是四类吸收机制，所以能跑，但不可长期托付。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:50-52]

### 五条协议各自对着一个具体失效模式

**Prompt 装配**处的失效是"模型忘了先干活、边做边报"：写进 system prompt 的要求，在几十次连续工具调用后会被稀释。解法是三道模型之外的检查——首条回复前插提醒、连续调用超阈值仍无消息时催汇报、turn 结束核对 SendMessage 是否真进了用户可见记录。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:27-31]

**记忆冻结**处的失效是长 turn 中途身份漂移：profile 与 memory 在同一 compaction epoch 内冻结，等于承认系统身份与过去事实不该在一次任务内变动，否则压缩一次助手就换了个人。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:31-31]

**持久会话状态**处的失效是重启即失忆、重试即重复：接受、发送、确认拆成三段，Ack obligation 落盘可 redrive，无法确认旧动作完成时要求用户重发而非假装成功；新消息可中断旧 turn，晚到 checkpoint 被 fence 丢弃。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:46-47]

**长循环步边界**处的失效是无限自嗨与资源失配：每轮重建 toolset 把能力变成本轮授权结果，执行器缺失即 fail closed；context 四通道分账，巨型工具日志因此吃不掉任务计划；Router 的 8 turn/step 上限则是反面样本——用硬截断代替循环治理。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:34-39]

Provider 绕过原生 Host 时，这些失效逐条兑现：无 prompt assembly，提示词退化；无 memory freeze，压缩改写身份；无 durable state，重启丢掉未完成的交付义务；无长循环，第 9 步后没人验收。旁路 provider 因此是"能用的适配器"，不是"原生 Agent 的简化版"。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:50-50]

### 持久云电脑与终端编码 Agent：延迟与状态所有权的置换

终端编码 Agent 用本地执行买低延迟：命令在用户机器上跑，状态所有权落在工作目录与 git，出错就重跑。持久云电脑把延迟藏进 Computer 画面——用户看到一块持续变化的屏幕而非一次调用等待；换来的是动作要跨 Host、box、页面三层校验，审批因此贵得多：动作、目的、box 与页面 display-state identity 一起入批，执行前重查，窗口一变旧卡即失效。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:42-44]

代价在顺序上：一致性优先于速度。把"读时刻的页面状态"当分布式事务处理，等于拒绝在未确认时落副作用；状态所有权挪到 Host 后，durable acceptance、generation fence、ack redrive 才成为必需——云电脑没有"重跑一遍"这条退路，这与 [[entities/durable-harness-wal-replay-vercel-2026|durable harness 的 WAL/replay 思路]] 同族。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:22-24]

但隔离边界没跟上：Electron 开了 context isolation、关了 Node integration，Chromium sandbox 却关闭并追加 no-sandbox。安全来自窄 bridge、协议校验与业务权限而非内核——持久云电脑把信任从 OS 挪到业务层，这既是它能跨 surface 的原因，也最需要审计。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:44-44]

### "产品运行时反推"能证明什么，只能推断什么

**能证明的**是契约层：模块边界、调用顺序、协议字段、fail-closed 分支与写死的常量（10k/5k 阈值、三次 nudge、8 turn 上限）；作者跑通的 typecheck 与十个契约测试覆盖的也只是这一层。**不能证明**的是服务端策略、真实线上阈值与正式打包下的一致性：作者自己标了环境与声明不符（Node 24.19.0 对 26.5），那组测试证明源码契约可运行，不是发布物已验证。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:52-52]

**只能推断的**是设计意图：阈值背后的产品判断、某道 middleware 为哪次事故而补，bundle 不会回答。前端更弱一层：发行包只有优化后的 renderer，可读前端靠字节锚点与协议行为恢复，不能当官方 React 源码。^[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026.md:19-19]

落点是：这类拆解产出**实现契约**，不产出**设计意图**。引用时把"代码里确实有这道检查"与"作者认为它在解决什么"分开写，否则一次高保真重建会被读成官方架构文档——这与 [[entities/claude-fable-5-prompt-leak-runtime-control-plane-vibecoder-2026|Claude Fable 5 提示词泄漏运行时]] 共享同一套证据纪律。

## 实践启示

1. **先列失败类，再定层数。** 每层要能一句话说出它吸收哪类错误，说不出来就是功能分组。

2. **把交付当 side effect。** 回复要有 acceptance / ack / redrive 三段；只留"调用成功"一个信号，重启后没人知道那句话出去没有。

3. **冻结身份，分账 context。** profile 与 memory 固定 epoch；语义、承诺、长期事实、视觉状态各走一条通道，任一条写坏不该拖垮其余三条。

4. **每轮重建能力集，缺件即 fail closed。** schema 不出现无权能力，executor 缺失直接拒绝，顺带省掉低频工具的上下文开销。

5. **审批绑定状态，不绑定人。** approval 带 Agent、tool call、动作、目标、epoch、内容哈希与页面 identity，任一变化旧授权作废，执行前重查。

6. **判断旁路是否等价，看它绕过了哪些吸收机制。** 别比 UI，看它有没有经过 prompt assembly、memory freeze、durable state 与长循环治理；缺一项就只是适配器。

## 相关
与 [[entities/grok-build-agent-kernel-vibecoder-2026|Grok Build Agent 内核]] 同作者同类型（VibeCoder 源码级拆解），但为不同产品（终端编码 Agent vs 持久云电脑 AI 队友）。与 [[entities/claude-fable-5-prompt-leak-runtime-control-plane-vibecoder-2026|Claude Fable 5 提示词泄漏运行时]] 同为"产品运行时反推"侧写。与 [[entities/harness-engineering|Harness Engineering]] 主题互链。→ [[raw/articles/grok-bot-agent-runtime-five-layer-vibecoder-2026|原文存档]]
