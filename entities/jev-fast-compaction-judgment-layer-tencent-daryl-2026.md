---
title: "Jev 快判断层与 fast-jev-compaction 上下文剪枝：把 Agent 的判断题从大模型里拆出来"
created: 2026-09-21
updated: 2026-10-20
type: entity
tags: [jev, system-one-model, context-compaction, fast-jev-compaction, agent-architecture, judgment-layer, probability-threshold, typesafe, tencent, rlcd]
confidence: 0.85
provenance_state: merged
sources: [raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026, raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026, raw/articles/ai-comes-for-the-if-statement-tunguz-2026, raw/articles/2400万人围观前openai研究员做了个闭嘴模型, raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25, raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12, raw/articles/agentjev-training-pipeline-datapai-2026-10-12, raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14, raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
---

# Jev 快判断层与 fast-jev-compaction 上下文剪枝

> 原文存档：[[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026|原文存档]]

## 摘要

腾讯技术工程（daryl，高级前端开发）对 TypeSafe AI System One Model「Jev」的工程实践长文：核心命题是 **Agent 里很多模型调用不是为了生成答案，而是为了替代码做一次判断**——Jev 不是"更便宜的 GPT"，而是 Agent 系统里缺了很久的**快判断层**（"不是 Agent 的大脑，而是 Agent 的反射神经"）。文章以 GitHub 项目 fast-jev-compaction 为切口做**逐文件源码级拆解**（collectToolCalls 配对 / fitState 八级降级链 / keepCall+keepResult 双 Noul 决策树 / 分批请求+失败回退），并自建可运行 Demo 实测：11/11 测试通过，压缩率 91.5%（10029→852 字符），关键事实（鉴权头/超时栈/文件定位）完整保留。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

## 核心要点

### Jev 定位：把生成问题改成判断问题

三原语（Choice 单选题 / Score 打分题 / Noul 判断题）与 GPT/Claude structured output 的本质差别：**不是能不能返回 JSON，而是从一开始就放弃自由文本生成，只做封闭输出空间里的概率判断**。模型输出从"文本"变成"可执行的概率判断"——`if (keepResult >= 0.5) { keepFullResult(); }` 直接进入代码分支。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 四层架构分工

| 层次 | 职责 | 典型组件 |
|---|---|---|
| 慢思考层 | 规划、解释、生成、复杂推理 | GPT / Claude / Gemini |
| 快判断层 | 路由、筛选、评分、门禁 | Jev / Jev-like 模型 |
| 确定性层 | 权限、状态、副作用、回滚 | 普通代码 |
| 兜底层 | 高风险或低置信度处理 | 人工 / 更强模型 |

生成负责表达，判断负责分流，代码负责执行。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 为什么需要快判断层：高频 Agent 的三层断裂

1. **生成成本和判断需求不匹配**——"这段日志还要不要留"是判断题不是作文题，为它调用大模型读完整上下文+生成解释+代码解析，链路太重；
2. **摘要会破坏可复核性**——工具结果是后续定位问题的证据（路径/错误码/栈信息/命令参数），摘要把这些精确内容变成"含义相近"的描述，而 Agent 下一步要继续执行动作不是理解大意；
3. **置信度没有进入代码分支**——通用大模型的"80% 把握"只是文本自我描述非校准概率；Jev 把概率变成接口返回值，按阈值分流（≥0.8 自动执行 / 0.5-0.8 保守处理 / <0.5 删除或回退）。真正撑爆上下文的往往不是用户或助手说的话，而是**工具结果**。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### fast-jev-compaction：不做摘要，只做保留决策

Claude Code 插件 + npm 库。核心约束：**用户文本和助手文本不改写，只处理工具调用和工具结果**（用户文本含原始硬约束、助手文本含已承诺计划，摘要改写易变味；工具结果占空间最大、最易过期、可重读重执行）。把上下文压缩拆成两个判断题：`keepCall`（工具调用本身还重要吗）+ `keepResult`（结果全文还需要保留吗）→ 三种动作：keep（完整保留）/ drop_result（保留调用截断结果）/ drop_call（一起删除）。与传统 summary 是两条路线：重要原样留下、不重要直接删、中间态保留调用截断结果——**不改写事实，只在"留多少"上做选择**。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### Demo 源码拆解（逐文件行级）

- **collectToolCalls**（src/state.js:80-123）：按 toolUseId 把 tool_use/tool_result 配对，避免孤立调用或孤立结果的危险状态；统一对象含 id/toolUseId/tool/input/callIndex/resultIndex/resultChars/isError/**pinned**（首条+最近 6 条固定保留，防刚发生的上下文被过早裁掉）；
- **state 构造**（src/state.js:146-183）：不把完整工具结果发给 Jev，只发短说明（`error, 830 chars (omitted)`）——"压缩器不需要读完整历史，只需要知道哪些历史值得继续被读到"；
- **fitState 八级降级链**（src/state.js:192-241）：full（输入≤1000 字符）→ inputs≤200 → inputs≤60 → texts abridged（头尾保留）→ old messages collapsed → old calls compacted → old messages left out → old calls merged；讲究顺序：先削细节再删内容、先处理旧消息再处理最近、先保护任务连续性再追求压缩率；stateStage 统计字段让压缩可观察（经常落到 left out = state 预算太紧需调策略）；
- **questionsFor**（src/jev.js:100-113）：一个问题只做一个判断——不问"调用和结果是否重要"混合题，拆成两个独立 Noul；
- **decideCall 决策树**（src/compact.js:131-149）：pinned 强制 keep；keepResult≥threshold → keep；keepResult<threshold 且 keepCall≥threshold → drop_result；都低 → drop_call；
- **batchCalls + Promise.all**（src/compact.js:86-114）：每批重复完整 state（成本升但判断同上下文），失败回退表（请求失败→回退 summary 或不压缩/格式异常→丢弃本轮/收益不足→保留原 transcript/state 无法 fit→调大预算/低置信度→保守保留）；
- **asker 抽象**：有 TYPESAFE_API_KEY 走真实 Jev，无 key 走本地启发式 mock——"模型可以替换，决策协议要稳定"。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### Demo 实测结果

样本为修复订单导出超时的会话（4 个工具调用）：t1 list_dir → drop_result（0.54/0.00，目录树可重跑）、t2 read_file orderExport.ts → keep（0.59/0.69，含 getAuthToken 与"不要破坏鉴权"约束相关）、t3 execute_command 失败日志 → keep（0.75/1.00，超时+authorization header）、t4 read_file 无关 legacy 文件 → drop_call（0.47/0.00）。压缩前后：10029→852 字符（-91.5%），完整保留 2、截断 1、删除 1——**上下文压缩可以从"总结旧历史"改成"判断保留价值"**。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 选型矩阵：Jev vs Structured Output vs 传统分类器

| 方案 | 适合位置 |
|---|---|
| LLM + JSON Schema | 低频复杂判断（需解释理由） |
| Tool Calling | Agent 主流程动作 |
| 传统分类器 | 稳定标签+大量标注（便宜可本地） |
| Jev | 高频局部判断（输出封闭、概率可用） |

选型判断六问：输出选项能提前枚举/是否高频/错误可兜底 → 适合 Jev；需自然语言解释 → LLM；长期稳定有标注 → 传统分类器；跨文件多步推理 → Jev 不适合单独承担。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 两个宣传误读与工程护栏

1. **"不会幻觉"实为类型安全非语义正确**——不会输出 schema 外内容（不会编 maybe_keep），但仍可能选错；
2. **"概率可信"尚不充分**——RLCD 训练细节与第三方校准数据不充分，不能直接把 0.9 当上线规则；正确姿势：shadow mode → 用业务样本统计各概率段真实正确率 → 再定阈值 → 高风险保守。
3. **风险分层护栏**：L1 只读（搜索/读取/分类/重排）可自动执行；L2 可逆（截断上下文/临时记录）低置信度保守保留；L3 不可逆（删除数据/扣款/关闭权限）**不让 Jev 单独决定**——"概率能帮代码分流，但不能替业务背锅"。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 社区项目方向表

浏览器自动化（browser-use/jev-ultrafast 页面元素编号选下一步）/ Agent 框架（vercel/eve 评估路由）/ 上下文剪枝（fast-jev-compaction）/ 代码审查（jev-review 拆是非判断+风险评分）/ 本地复现（SemIf/kev/laya 开源小模型复刻 Jev-like 接口）/ 数据库扩展（pg-jev SQL 语义判断条件）/ 游戏仿真（Doom/Mario/Snake 每 tick 选动作）。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

## 深度分析

### "判断"作为独立层的组件化意义

文章最有价值的抽象：把过去散落在 Prompt、规则引擎、小模型和主模型里的判断能力单独抽成一层接口。三个变化：效率（不再为每个小判断启动完整生成链路）、能力（路由/筛选/评分/门禁沉淀成统一接口）、模式（Agent 从"全靠大模型思考"变成"生成、判断、执行分层协作"）。Jev 的技术零件（分类/约束输出/logits 打分/概率校准/判别式路线）以前就有，新意在于组合成清晰开发者接口。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 压缩路线的方法论意义

fast-jev-compaction 展示的"判断保留价值"路线与"摘要改写"路线的根本分歧：后者用生成能力换 token，风险是把关键事实改写成"差不多"；前者把压缩变成筛选任务，精确性优先，代价是要维护判断接口与阈值。与 [[concepts/loop-engineering-methodology|Loop Engineering 方法论]] 中"验证结果+写入状态"两动作呼应：状态必须在对话之外且不被改写。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

### 与同主题解读的互补

[[raw/articles/jev-system-one-model-15-projects-datafun-2026|DataFun Jev 15 项目生态综述]] 提供生态全景（239 Build/成本数字），[[raw/articles/jev-rlcd-decision-function-feixue-ai-2026|飞雪谈AI Jev 机制解析]] 提供 API 机制+RLCD 训练原理+真实调用案例；本文提供**源码级工程实现**（逐文件行级拆解+自建 Demo 实测）——三文构成 Jev 主题的生态/机制/工程三视角。^[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026.md]

## 实践启示

- ✅ **快判断层引入判据**：输出空间固定+调用频率高+需语义理解的小判断 → Jev-like；每天几十万次调用才有明显意义，每小时几十次用通用大模型即可
- ✅ **压缩任务拆判断题**：keepCall/keepResult 两 Noul 替代混合问题，概率组合映射三种动作，可测试可回退可统计
- ✅ **state 构造原则**：给判断模型的信息只保留判断所需（短说明代替全文），压缩器不需要读完整历史
- ✅ **降级链设计**：预算不足时逐级降级（先削细节再删内容、先旧后新），stage 统计让退化可观察
- ✅ **概率上线流程**：shadow mode → 概率段真实正确率统计 → 定阈值 → 高风险保守；L3 不可逆动作不让判断模型单独决定
- ✅ **asker 抽象**：模型可替换，决策协议要稳定

## 相关实体

- [[entities/agent-harness-evolution-from-llm-call-to-harness-tencent-2026|Agent Harness 演化论]] — 同源腾讯技术工程，Harness 六层演化视角
- [[entities/cursor-router-production-model-routing-2026|Cursor Router 模型路由]] — 快判断层在 LLM 间路由的同类位置
- [[entities/pyrodash-token-level-small-large-collaborative-inference-2026|PyroDash token 级协同推理]] — 小模型判断+大模型生成的级联分工
- [[entities/query-aware-rag-cost-compression-pattern|Query-Aware RAG 成本压缩]] — 小模型过滤降成本的同型模式
- [[entities/a-missing-layer-in-agentic-systems|A Missing Layer in Agentic Systems]] — "缺一层"的同类架构论证（HITL 层）

→ [[raw/articles/jev-fast-compaction-judgment-layer-tencent-daryl-2026|原文存档]]

## 第 2 来源 — 腾讯技术工程（mason）Jev 上手 Demo：接口三型与「判断/生成」双模型的延迟证据

来源：[[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026|聊聊最近爆火的 Jev 模型，到底是个啥？]]（腾讯技术工程，mason，Principal Research Leader，2026-09-21）。同为腾讯技术工程对 Jev 的解读，但取的是**上手实测视角**：作者申请 API Key 后用 CodeBuddy 做了一个网页小游戏 demo，再横向看社区 demo。v×c≈36（双 API down 走 canonical heuristic，落 35–48 区间），与本文 70%+ 主题重叠（同一模型、同一「判断层从生成层拆出」命题），互补角度 ≥3 ⇒ 按 MERGE 矩阵并入本页作第 2 来源。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]

**互补角度 5 条**：

1. **接口三型的调用形态复述**：Jev 三种接口被落成「问什么 / 返回什么」表——Choice（玩家这句话在叫谁？→ 一个选项 + 各选项概率 + confidence）、Score（这段内容风险处于哪个等级？→ 按定义等级评分 + 概率分布 + confidence）、Noul（这条请求需要转人工吗？→ 取 0–1 的是非概率，不单独返回 confidence）。游戏场景里每一步的提问是「我现在有多少血，怪物和金币在哪里，出口在哪里，目标是拿到金币并活着出去，现在应该按哪个键」，可选项**预先枚举**为上/下/左/右/攻击/喝药，模型返回其一，执行后再把新局面发回——本页「三原语」小节的一手调用实例。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]
2. **概率 ≠ 置信度 ≠ 胜率 的语义区分（对本页「概率可信尚不充分」护栏的直接补充）**：demo 第一步返回「攻击」概率约 95%，界面置信度约 93%，作者明确点出这是两个不同的数，且**都不是「这局有多大胜率」**——清晰划开了「分类概率」「模型自报置信」与「任务成功率」三者，正是概率阈值上线前必须做的语义校准。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]
3. **Browser Use jev-ultrafast 的具体延迟证据**：页面先被整理成带编号的元素，Jev 决定操作什么、点哪个目标，需要填城市名时再调用生成模型产生文字；作者记录的一次航班查询用了 **7.073 秒，其中 17 次 Jev 请求 + 2 次文字生成调用**（开页与最终独立校验不计入）。本页「社区项目方向表」原先只列了该项目名称，此处补上数量级：高频局部判断确实由小判断模型承担，生成调用被压到 2 次。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]
4. **分工叙事的具体化**：地图、伤害、碰撞由普通游戏代码处理，每一步按哪个键由 Jev 返回——「**Jev 是拿手柄的玩家，普通代码是游戏本身**」；它看到的是状态描述而不是屏幕截图，也没有人工写好的通关路线。这正是本页四层架构分工（慢思考/快判断/确定性/兜底）在具体项目里的落点。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]
5. **对照实验的方法论诚实边界**：作者明确写出「我们已经搭了相同地图、相同规则的对比版本，但目前没有完成有效的双模型实测，所以这里不放胜负和性能结论」，并用一次低血量测试（20 点血时先攻击，被反击后剩 5 点血）说明「这个动作是不是最优要看后续局面，不能只凭没喝药下结论」。这是与本页「两个宣传误读与工程护栏」同向的方法论自律，也解释了为何上一条延迟数字可用于工程估算、却不可用于「Jev 比 GPT 更会玩」这类结论。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]

**边界**：该文为个人上手 demo 记录（6 次真实 API 调用、单局游戏 + 一次航班查询），不构成基准对比；作者本人亦声明演示刻意放慢、不能说明响应时间。^[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026.md]

→ [[raw/articles/jev-model-demo-browser-use-benchmark-tencent-mason-2026|第 2 来源原文]]

## 第 3 来源 — Tomasz Tunguz（Theory Ventures）：If-Statement 经济学——harness margin 与生产环境 98 封邮件实测

来源：[[raw/articles/ai-comes-for-the-if-statement-tunguz-2026|AI Comes for the If Statement]]（Tomasz Tunguz, Theory Ventures GP, 2026-09-22）。LLM 评分 v=7/c=7/stars=4（v×c=49, ark glm-5.3-flash）。与前两源（技术解读 + 上手 demo）不同，Tunguz 取的是**投资人经济学视角**：把 Jev/SemIf 这类 machine-native 判断模型放进「软件原语特化 → harness 毛利扩大」的框架里。70%+ 主题重叠（同一模型家族、同一「判断与生成分离」命题），互补角度 ≥3 ⇒ 按 MERGE 矩阵并入本页作第 3 来源。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]

**互补角度 4 条**：

1. **98 封生产邮件线程的对照分类实测**：同一数据集上，生产环境生成式 LLM 分类器 47%（46/98），Jev 80%（78/98），本地 SemIf 82%（80/98）——判断模型在生产邮件分流场景**同时更准更便宜**的一手数字；另有 31 封线上邮件试点，本地判断模型处理 8 封零错误、其余安全升级到 frontier 模型。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]
2. **成本量级：76x–209x 价差**：Jev 定价 $0.042/M input + $0 output，对比 Sonnet 级 $3/$15——按 2000 input + 60 output 的典型分类调用算 82x 便宜；TypeSafe 自己的 workflow evals 给出 $0.0004 vs $0.0304/$0.0836（76x–209x）。本页成本论证的最sharp的一组数字。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]
3. **SemIf 机制描述（本页开源复现条目的原理补全）**：不做自回归 token 生成，attention 只跑一遍，直接从输出 logits 评估候选选项，完全跳过多层 FFN 与 decode 步骤——与 [[entities/pyrodash-token-level-small-large-collaborative-inference-2026|PyroDash token 级协同推理]] 的「跳过生成路径」同型。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]
4. **Harness margin 命题（经济学视角独有的战略判断）**：AI 的分工正在两极化——frontier 模型做发现与架构、特化模型做生产；harness 的价值是把执行成本压下来而不牺牲准确率，特化原语进一步扩大 harness 的毛利空间。「harnesses are about to capture a lot more margin」是 [[entities/agent-harness-evolution-from-llm-call-to-harness-tencent-2026|Agent Harness 演化论]] 的工程叙事之外的资本视角印证。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]

**边界**：Tunguz 是 Theory Ventures GP，文内有基金立场（投资叙事框架）；邮件分类实验为作者自测（hand-verified 但非独立复现），Jev 定价被 TypeSafe 自己标注"may be subsidized"。^[raw/articles/ai-comes-for-the-if-statement-tunguz-2026.md]

→ [[raw/articles/ai-comes-for-the-if-statement-tunguz-2026|第 3 来源原文]]
## 相关页面

第 4 来源（机器之心报道，RLCD 训练方法 + Diogo Almeida 背景 + 官方/外部性能数字）：[[entities/jev-rlcd-machine-heart-report-2026|Jev 与 RLCD：机器之心报道]]

## 第 5 来源：Jev 前史——港中文 TimePrism「从采样到场景」（机器之心 2026-09-25）

机器之心对 Jev 现象的**前史考证**：Jev 受关注的三个设计特征（显式概率 / 并行输出 / 面向决策）并非 TypeSafe 首创——港中文徐强团队 2025-09-24 的论文《From Samples to Scenarios: A New Paradigm for Probabilistic Forecasting》（arXiv 2509.19975，已中 ICLR 2026）在概率预测领域已提出同一命题。^[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25.md]

核心增量与互补角度：

- **␻samples→scenarios 范式**：TimePrism 把学习目标直接写成 {场景，概率}——一次前向并行输出整组可能未来及各自权重，取代「反复采样轨迹→估计分布」的传统路径；决策系统可对各场景直接算预期成本。^[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25.md]
- **极简架构自证范式**：核心可学习结构仅三个并行线性层（两分支生成场景 + 一分支学习概率），在五个基准十项比较中九项最佳，胜过扩散/Flow/Transformer 基线——与团队 2023 年 DLinear《Are Transformers Effective for Time Series Forecasting?》（引用 6000+）「简单结构+正确目标函数」的路线一脉相承。^[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25.md]
- **Jev↔TimePrism 收敛判读**：两者共同点是根据决策需求重设计输出——概率成为模型与决策系统之间的**接口**；差异在 Jev 面向给定问题作判断（选项由开发者定义），TimePrism 连候选未来也交给模型学习。^[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25.md]
- **推广方向**：机器人轨迹比较、规划多后继状态、资源调度等「多候选结果+概率」任务都可能受益于该输出形式。^[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25.md]

与本 entity 的 RLCD 校准叙事互为印证：Jev 的「typed probabilistic decisions out」在概率预测领域有同构先例，「AI 直接输出概率接口」正在从预测任务向 Agent 判断层扩散。

→ [[raw/articles/jev-shuaping-prehistory-timeprism-samples-to-scenarios-machine-heart-2026-09-25|原文存档]]

## 第 6 来源：AgentScope Java 官方集成——11 组件全决策流程（阿里云云原生 2026-10-12，v=7/c=9/63 SUPP）

阿里云云原生第一方集成教程：AgentScope Java 经 `agentscope-extensions-jev` 正式接入 TypeSafe JEV，把本 entity 的「判断层」从单一压缩场景扩展为**执行前→执行中→输出→交付四阶段的 11 组件全家桶**。^[raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12.md]

- **契约层（库内零覆盖）**：SystemOneRequest = state + 按 ID 命名问题；概率语义三条边界（0.92 是概率判断非操作成功、非业务实测准确率）；JevJudge 状态四分 **PASS/FAIL/INCONCLUSIVE/ERROR**——证据不足与调用故障不与判断失败混同；运行模式 **OFF/SHADOW/ENFORCE** 按组件独立启用，SHADOW 下主模型仍收到原工具目录（先观察后采用）。^[raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12.md]
- **11 组件**：执行前 JevToolSelection（多选+数量限制，"明确都不适用"≠"判断失败"，RECENT_WINDOW 8 条/8000 字符）/ JevPhaseRouting（阶段候选池按能力/容量/配额排除）/ JevTeamPlanner / JevAutoMode 执行防护（ENFORCE 拒绝不派发，判断通过≠授权）；执行中 JevContextCompactor（与本 entity 第 1 来源 fast-jev-compaction 同题的官方化版本：归档先行/恢复不重执行/截短保前缀≠摘要）+ JevMemoryGate（持久价值+冲突评审，压缩不自动转长期记忆）+ JevEvidenceProcessor（区分提示注入/答案证据/前提冲突——「退款需审核」否定前提却是关键证据）；输出 JevResponseMiddleware（失败才有限次纯文本修订，不重跑工具循环）；交付 JevSupervision（OFF/SHADOW，宿主验证证据要求——自称完成不构成交付）+ JevTraceEvaluator（指标保留分类/分数/状态，跳过/弃权/错误不混成低分）+ JevCodeReviewer/JevBrowserNavigator（只读，独立验证通过才 VERIFIED）。^[raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12.md]
- **与本 entity 互补**：daryl 的 fast-jev-compaction 是社区单点实践（压缩率 91.5%），本文是平台方把 JEV 编排为 Harness 中间件标准的完整接口面——「判断层」进入框架级采纳阶段。^[raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12.md]

→ [[raw/articles/agentscope-jev-harness-integration-aliyun-2026-10-12|原文存档]]

## 第 7 来源：AgentJev 开源复刻训练全流程（数据派THU 转自 Datawhale 2026-10-12，v=7/c=6/42 SUPP）

判别式决策模型的**训练侧**补强（前 6 来源均为推理/生态视角）：AgentJev = Jev 的开源社区复刻，Qwen3-0.6B 底座（去语言模型头）+ 决策候选头（投影层+两层轻量 Transformer+线性打分层），网络末层直接输出候选标量得分→归一化概率，一次前向数十毫秒。^[raw/articles/agentjev-training-pipeline-datapai-2026-10-12.md]

- **训练机制（零覆盖）**：全参微调（非 LoRA），阶梯学习率骨干 1e-5/候选头 1e-4（候选头从零初始化需高一个数量级）；损失 = soft cross entropy + 0.1×Brier（抑制过度极端置信度）；数据集 LocalLLaMA/typed-decisions（案例制，state+questions[boolean/choice/score]+gold 目标分布，factors 字段显式过滤防走捷径）；**案例级切分防泄漏**（同案例多题不得跨集）+ state 文本哈希查重断言。^[raw/articles/agentjev-training-pipeline-datapai-2026-10-12.md]
- **三阶段顺序不可颠倒**：检查点选优（只看 dev 未校准软交叉熵）→ 温度校准（calibration 集 81 候选温度拟合，实测 1.0-1.0718≈天然对齐）→ 测试集终评 78.05%（vs 官方 79.25%，差 1.2pp 同区间）；实测 DGX Spark 600 步 49 分钟/峰值 34GB。服务化后以胜出置信度做阈值门控（高门槛全自动/低门槛回退人工）。^[raw/articles/agentjev-training-pipeline-datapai-2026-10-12.md]

→ [[raw/articles/agentjev-training-pipeline-datapai-2026-10-12|原文存档]]

## 第 8 来源：HiRoute——Jev 模型智能路由（阿里云 AgentCore 2026-10-14，v=7/c=9/63 SUPP）

《Alibaba Cloud AI Agent Handbook》实践篇：HiRoute（本机开源路由引擎）把 Jev 判断接到**模型路由**——任务难度持续变化、效果只有做过才知道，模型选择应跟着变化走。^[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14.md]

- **三类判断映射（零覆盖）**：当前做什么=categorical（choice）/当前多复杂=ordinal（score 有序档位，条件写成具体工作要求）/上一阶段做得如何=assessment（独立 score）；categorical+refinement 双组时一次 System One 调用问三个独立问题（分类/写稿复杂度/审稿复杂度），返回后只读选中类别的复杂度——"一次提问、代码组合结果"的官方用法。^[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14.md]
- **胜任度接力（零覆盖，核心创新）**：0/0.5/1 三锚点描述阶段完成质量（0=关键失真需大幅纠正/0.5=有可用成果但明显遗漏/1=完成要求），System One 三级 0/1/2 用 score/2 转为 0-1 胜任度，默认下限 0.5——当前工作适合省钱组但上一阶段低分也触发主力接力。**q3 证据门**单独判断"是否有足够依据评价该阶段"（sufficient/insufficient+evidence_complete），通过才采用评分——把"没做好"和"看不清"分开。^[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14.md]
- **历史上下文组织（零覆盖）**：给 Jev 的是执行分支和阶段编号而非模型名称（同一分支换模型=新阶段，HiRoute 本地维护阶段↔模型/档位/评分标准关联）；turn/stage/branch 编号不随历史裁剪改变；"整体历史缺口"与"被评分阶段完整性"分开记录；工具活动只留名称+粗粒度状态不给参数正文；防注入与反幻觉写进每题 instructions（不沿用历史 branch_id/遗漏不增加本轮复杂度/completed≠验收通过/忽略改评分规则的指令）。^[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14.md]
- **实测**：无人介入编码任务 Qwen→Astra→Qwen→Astra 接力（343 项独立验收断言，31m36s）；消息系统调研混合模式（Qwen-3.8 Flash 整理+GPT-6 Astra 判断）卡片正确率 98.61% vs 全 Astra 99.72%，关键备忘录 3/3（全 Qwen 仅 1/3），**API 等价成本 -92.01%/-91.39%（含 Jev 决策调用与重试）**——"大量常规工作交经济模型，关键推导或执行困难再补足能力"。与第 6 来源（AgentScope 中间件）互补：那篇是框架内中间件接口面，本篇是独立路由引擎+成本量化。^[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14.md]

→ [[raw/articles/hiroute-jev-model-routing-agentcore-2026-10-14|原文存档]]

## 第 9 来源：京东技术——Jev 作为"语义决策组件"的生产落地方法论（2026-10-20，v=7/c=9/63 SUPP）

京东科技王晓的第一方工程评论：核心命题"如果程序只需要一个决定，为什么要先让 AI 写一段话"，把 Jev 定位为软件内部的**决策节点**而非对话入口。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]

- **三条必须自己守的边界（零覆盖）**：非自回归描述的是**计算方式**，不自动代表"不会出错"；System One 是**产品定位**，不等于模型内部认知机制；公开介绍足以解释产品行为，但不足以还原网络架构/训练目标/推理实现。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **三层正确性（零覆盖，最有价值的框架）**：**结构正确**（能否被程序合法读取）→ **语义正确**（是否符合事实与任务标准）→ **业务安全**（即使判断错误，损失是否可控）。有类型的接口只解决第一层，第二层靠评测、第三层靠系统设计——"不能把'不会生成答案集合之外的文本'扩大成'不会发生事实错误或业务误判'"（例：ALLOW/REVIEW/BLOCK 返回 ALLOW 类型合法但业务上可能致命）。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **Amdahl 换算（零覆盖）**：宣传的 200×/400× 是厂商基准（LangChain/Browserbase 转述口径为 20–200× 速度、40–400× 成本）；若决策只占链路 20% 耗时，100× 的决策加速只带来 `1/(0.8+0.2/100) ≈ 1.25×` 整体加速——应比较"每完成一个合格业务结果的成本"，而非"每次调用价格"，并看 P50/P95/P99 与整链路成功率。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **概率校准要落到指标（零覆盖）**：`P=0.95` 是个体上的一个数，校准看的是**群体频率**——落地需可靠性图、Brier Score、分组校准、覆盖率-错误率关系、分布漂移后的表现；概率的用途是决定"自动执行/升级/拒答"。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **决策契约六要素（零覆盖）**：输入 / 输出标签 / 每个标签的业务含义 / **弃权**（何时算无法判断）/ 回退（转给谁）/ 后果（误判损失）——比优化提示词更重要。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **多方基线（零覆盖）**：至少并列**确定性规则、传统分类器/小模型、LLM 结构化输出、Jev** 四类，否则只证明了"原来过度使用大模型"，没证明 Jev 最优。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **影子运行与放量路径（零覆盖）**：离线评测 → 影子运行（不影响真实决策）→ 低风险灰度 → 分类别扩大 → 持续监控复评；必须记录模型版本、问题模板、答案集合、策略版本与最终业务结果，否则无法定位退化发生在哪一层。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **四组件分工与接口契约演进（零覆盖）**：生成模型负责表达、决策模型负责选择、检索系统负责证据、确定性程序负责约束执行；API 从 `classify(): Category` 走向 `Decision<T> = {value, confidence, policyVersion}`，调用者必须处理"**合法但不够确定**"这一新状态；测试从单点断言扩展到统计保证（分组漏报、覆盖率-错误率、旧指标回归）。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- **四个试点场景与各自陷阱**：①客服/工单分流（无"其他/无法判断"选项会强制归类）；②Agent 动作审核与工具路由（**答案空间受限不解决提示注入**，且只能作辅助层，确定性权限/金额/范围校验不可省）；③RAG 相关性筛选与候选排序（分数未必可跨查询比较，仍需专门检索评测）；④研发局部评审（补充而非取代编译器/测试/安全扫描/代码评审，"模型认为风险低"不是跳过测试的理由）。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]
- 与既有来源的关系：第 1 来源（腾讯 daryl）是**源码级**证据（fast-jev-compaction 逐文件 + 91.5% 压缩率实测），本来源是**治理级**证据（如何把决策点接进生产而不被宣传数字误导），两者构成"实现 → 落地"的两端；三原语 Choice/Score/Noul 已被本文档覆盖，故不计入增量。^[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20]

→ [[raw/articles/jev-system-one-decision-component-jdtech-2026-10-20|原文存档]]
