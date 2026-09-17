---
title: DECO — 腾讯数据工程 Agent Hook 护栏层实践
created: 2026-07-16
updated: 2026-09-17
type: entity
tags: [agent, hook, governance, harness, tencent, data-engineering, hitl, guardrail]
status: verified
confidence: 0.95
provenance_state: extracted
sources: [raw/articles/deco-agent-hook-governance-tencent-2026-07-16]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> 腾讯 DECO（Data Engineering Agent 引擎）的护栏层实践，用 Agent 框架的 Hook 切面在代码层确定性兜底三类问题：长文本偷懒、越权操作、上下文失忆。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

## 三类治理问题

腾讯 DECO 团队识别出 prompt 无法管控的三类 LLM 行为：^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

1. **LLM 偷懒** — 长 SQL/ETL 脚本被截断、占位略写、复印式重写到 token 耗尽
2. **越权操作** — 模型无法区分"查询"和"发布"的可逆性差异
3. **上下文失忆** — 模型跳过"看起来不必要"的检查步骤

## Hook 链护栏体系

基于 Agent 框架的 Callback 切面（beforeTool/afterTool/beforeModel/afterModel），DECO 挂了十余个 Hook。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### 长文本完整性护栏（读写两侧 Offload）

核心方案：LLM 永远不直接接触脚本全文。两端都用 Hook 拦截 + 沙箱文件做"中转站"。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

- **拉取侧 Offload（afterTool）**：Hook 拦截含 scriptContent 的响应，全文写入沙箱只读快照，替换为引用句柄
- **写回侧 Onload（beforeTool）**：Hook 从文件读回全文覆盖入参

效果：修改任务工具调用输出 token **直降约 90%**，复印自截断从近 100% 物理消除。

### 危险操作 HITL（beforeTool Guard）

配置驱动的**危险工具守卫**：每个危险操作配置 required-state key 和确认对话框，没拿到用户明确授权，工具在框架层物理走不通。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

确认框支持多选项 + 带输入控件（审批人、回刷日期），变更预览在确认前展示。

### 上下文联动闭环（Hook → state → Attachment）

范式：**Hook 采集事实 → 写 state → Attachment 注入下一轮 prompt**。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

- **RiskAnalysisHook**：Agent 改表后在 afterTool 触发下游风险分析，结果自动注入下轮
- **PythonImageHook**：Python 脚本产出的图表自动发现，生成预签名 URL 供前端渲染

## 行业定位

DECO 的护栏体系与主流框架对比：^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

| 能力 | 框架原生 | DECO 差异 |
|------|---------|----------|
| 读侧 offload | ADK ArtifactService, LangGraph offload | **写侧 onload** 自研（scriptFilePath 协议） |
| HITL | ADK ToolConfirmation, LangGraph HITL | 多选项+输入框+变更预览+配置驱动 |
| 上下文断裂 | ADK artifacts（被动 load） | Hook→state→Attachment **主动 push** |

## 深度分析

三类问题表面上是模型能力问题，实质上是把边界交给了一个概率系统执行：模型被优化为"用最短路径达成目标"，而省略、跳过检查、抢先调用工具恰恰都是最短路径。DECO 的做法把这三种失败模式各自映射到一条确定性的代码路径——偷懒对应长文本完整性护栏，越权对应 HITL 守卫，失忆对应上下文闭环——于是"该怎么治理"不再依赖模型是否读懂了警告。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### prompt 层约束为何必然失效

⚠️ 式禁令依赖模型在生成的那一刻仍把规则当作硬约束，但长文本任务里的偷懒是 token 预算压力下的必然产物：当输出逼近长度上限，模型倾向于用省略号、占位注释或复印式重写来"完成"格式上的任务，此时的失败不是不懂规则，而是规则在成本面前被折价。越权同理——查询与发布在模型看来都是"调用工具"，可逆性差异并不存在于它的表示里。因此 prompt 只能定义意图，边界必须由外部代码持有：能用确定性兜底的，就不该留给模型自觉。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### Hook 切面是 harness 的控制面

beforeTool/afterTool/beforeModel/afterModel 这些切面真正的价值不在"能插代码"，而在插入位置固定、触发条件确定，且推理侧对此无感知——ReAct 循环照常运转，护栏在旁路完成拦截、改写与注入。十余个 Hook 分散在不同切面，却不改变 Agent 的推理结构，实现基础设施与推理解耦。这与 [[concepts/harness-engineering-framework|Harness Engineering]] 的主张一致：harness 提供确定性的输送与约束，模型只负责在约束内做决策。代价是复杂度从推理层搬到了框架层——切面顺序、失败语义与相互干扰都必须被显式设计。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### 读写两侧 Offload 的 token 经济学与降级取舍

只做读侧 offload 只解决一半问题：脚本进上下文时被换成了引用句柄，但模型一旦决定改代码，仍要把全文重写回工具入参，于是每一轮都在重传整段 SQL。DECO 在写回侧再做一次拦截，从沙箱读回全文覆盖入参，使模型只需产出 str_replace 式的小步变更——token 直降约 90% 并非来自压缩，而是来自把"整段重写"这个动作从工作流里删掉。配套的两个权限语义同样关键：只读快照与工作副本分离，强制模型必须显式 copy_file 才能编辑，避免它在快照上直接动手；而落盘失败时回退为原内容继续执行，则是一处明确的取舍——宁可承担一次完整性风险，也不让护栏本身变成阻塞点。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### HITL 的机制化：让未授权调用物理走不通

"在 prompt 里请求用户确认"和"在框架层校验确认"不是同一量级的方案。DECO 把危险工具写成配置：工具名、required-state key、确认对话框的选项与输入控件全部声明式描述，状态前置条件不满足时调用直接中断，无论模型是自作主张还是被上游内容诱导。多选项加输入框（审批人 RTX、回刷日期）让确认动作本身承载业务参数，而不是一个布尔开关；变更预览则在用户决策之前把影响面摆出来。这套设计把安全属性从"模型愿意问"改写成了"不问就走不通"。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

### 主动 push 修的是时机，不是内容

ADK 一类的 ArtifactService 擅长被动 load——模型需要什么就去取，前提是它知道自己需要。上下文失忆的病根恰恰是模型不认为自己需要某项信息，所以被动语义治不了这个病。DECO 的 Hook→state→Attachment 拆成两层：采集由工具调用确定性触发，注入则刻意推迟到下一轮，既不污染当前轮推理，又保证信息出现的时间点正好是模型要下结论的时候。风险面在于 scriptFilePath 这类以框架协议、工具声明与配置格式承载的机制，会把治理逻辑绑进单一生态——换框架意味着整条护栏链重写，这是方案的通胀成本。^[raw/articles/deco-agent-hook-governance-tencent-2026-07-16.md]

## 实践启示

1. 把"禁止"从 prompt 搬进代码。凡是可判定的约束（上下文长度、工具名、状态 key）一律做成拦截器，只在确实无法判定时才交给模型权衡。
2. 长文本任务默认走 offload + onload：读侧替换为引用句柄，写侧支持小步编辑（str_replace），并显式区分只读快照与工作副本，编辑必须经 copy_file。
3. 危险工具用声明式配置登记：工具名 + 前置状态 key + 确认选项与输入控件，让未授权调用在框架层走不通，而不是靠模型礼貌询问。
4. 护栏必须自带降级语义：落盘失败、单条解析失败等异常应能局部退回原行为，仅该条降级，避免护栏自己成为新的单点阻塞。
5. 上下文补充走"确定性采集 + 延迟注入"：事实在 Hook 内采集写入 state，留到下一轮再注入，不要把注入时机提前到当前轮。
6. 评估可移植性成本：scriptFilePath 式的框架协议换来短期实现便利，长期锁定生态，落地前先确认退出路径。

## 关联条目

- [[entities/harness-engineering-alibaba-java-case-study|Harness Engineering 阿里 Java 实践]] — 企业级 Harness 工程另一视角
- [[entities/tencent-hunyuan-hy3-preview-hopper-inference-optimization|腾讯混元 Hy3 Agent 产品]] — 腾讯同系的 Agent 产品发展
- [[entities/tencent-k8s-ray-ai-workload-scheduling|腾讯 K8s + Ray AI Workload 调度]] — 腾讯另一生产级 AI 系统实践

## 退出

→ [[raw/articles/deco-agent-hook-governance-tencent-2026-07-16|原文存档]]
