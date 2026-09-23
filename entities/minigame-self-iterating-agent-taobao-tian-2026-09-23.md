---
title: "淘天小游戏自迭代 Agent：全链路生产-运营-迭代闭环"
created: 2026-09-23
updated: 2026-09-23
type: entity
tags: [ai, agent, multi-agent, self-iteration, game-generation, langgraph, claude-agent-sdk, level-generation, memory-system, ab-testing, taobao, tmall, first-party, closed-loop]
sources:
  - raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23
confidence: 0.9
provenance_state: extracted
---

# 淘天小游戏自迭代 Agent：全链路生产-运营-迭代闭环

## 核心命题

淘天集团营销&交易技术团队（作者辰霁、成禹）在淘天互动业务中落地的自迭代 Agent 系统，解决的核心问题是"**单点 AI 提效无法带动整体业务提速**"——即使 AI Coding、生图、关卡生成各环节已明显提效，上线一款小游戏仍需 2~3 周（从创意到原型最快 6 天），运营侧则面临经验难复用、三级数据（场域/游戏/关卡）分析不精细、结论缺少验证、归因缺乏统一上下文四大痛点。系统的分工原则：**人定义北极星指标与行动边界并审核关键方案，AI 负责策略、开发与数据归因**。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

## 五层全链路架构

- **L1 数据感知层**：把前一天的数据、变更记录和游戏代码整理成可信上下文
- **L2 运营 Agent**：判断当天最值得改哪款游戏，产出带预期的待评审方案
- **L3 生产 Agent**：人工审核后，将方案变成策划、美术、音效、代码和关卡产出
- **L4 游戏容器与模板**：可上线代码的执行环境
- **L5 运营 Agent（门禁与回收）**：设置门禁、沉淀知识，AB 实验到期时回收结果；完成后回到 L1 开始下一轮迭代^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

关键设计决策是**判断与执行分离**——不做包揽一切的"超级 Agent"：生产 Agent 接收已确认方案完成生产；运营 Agent 负责归因、优先级判断和"改什么"。两者职责隔离但可互相查阅上下文。生产闭环解决上下文流转（调研/方案/资产清单/关卡结构进入全局状态，用户中途改需求时由 Producer 统一判断影响范围并逐级路由重做）；迭代闭环处理数据驱动决策（感知→对齐目标→拆解→召唤专家→产出方案→读取 AB 回流→沉淀经验七步）。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

## 生产 Agent：Producer 统一调度 Multi-Agent

两大技术栈协同：**LangGraph** 负责全局流程编排与多 Agent 协调；**Claude Agent SDK（内部代号 tarot-code-agent）** 负责远程容器中真实代码生成。以 Producer（制作人）为核心调度节点，协调 6 个专业 Agent：Gameplay Research → Planner → Art Designer → Game Coder → Level Designer → Level Coder。Producer 不直接生产内容，只理解需求、规划链路、调度 Agent，中途需求变更时动态调整后续计划（如"新增一种敌机"→先策划更新资产清单→再生成美术→最后代码开发）。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

### 各环节针对性设计

- **玩法调研——以图代言**：不要求用户文字精确描述（"消除类游戏"可能指糖果传奇或俄罗斯方块），而是先搜索、后选图，用户"指"出感兴趣的截图，多模态模型从视觉提取核心玩法机制作为搜索关键词定位攻略，并筛选 2-3 张美术参考图贯穿后续生图流程。
- **策划方案——信息密度最大化**：GDD 每一句话都必须能被下游 Agent 直接消费，"如果一段描述不能映射到具体的开发动作，就不写"。区分程序资产（实体/状态机/特效）与素材资产（立绘/背景/UI/音频），先定义全局美术风格约束避免风格漂移返工。
- **美术资产——生产线式生图 Workflow**：风格一致性三层锚点（Base 调研参考图 / Style 首张生成图作锚点 / Entity 游戏对象标准态）+ 页面框架图合成后多模态布局分析输出结构化描述（Coder 读到布局意图而非零散图片）+ 四种执行模式（全量/局部重生/续生失败/咨询）由 LLM 意图分类。每张图走 Gemini 生图 + 抠图；音效用 ReAct 模式 LLM 自主规划清单，Tool Use 调用可灵 AI 生成，按 BGM/交互/状态/UI 分类输出。
- **关卡设计——设计关卡工厂而非关卡**：不逐一设计每一关，而是定义数据结构、生成算法、验证策略和难度曲线。核心是让每一关为**确定性产物：算法 + 参数 + Seed**，由此获得三项迭代能力——可溯源（Seed+难度因子精确复现问题关卡）、可改造（LLM 拿到真实生成代码，提案可明确到参数及目标值，改后 Seed 逐关 diff 回归）、可验收（蓝图与运行时复用同一条生成链路）。关卡方案严格限定 5 章（gameType/数据结构/生产方式/验证方式/难度曲线）。
- **Game Coder & Level Coder——远程容器编码**：唯一需要真实开发环境的 Agent，读写文件、装依赖、起开发服务器，由三层容器架构支撑。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

### Workflow 与自主推理的融合

纯 Workflow 每步固定灵活性差，纯自主 Agent 不可控且 token 消耗大。平衡点：**LangGraph 管"可控的流程骨架"**（主干确定，阶段间流转/审核/暂停恢复由结构硬性保证，模型跑偏也不失控）；**节点内部充分自主**（Planner 自定 GDD 章节深度、Art Designer 自析依赖顺序）；**Human-in-the-loop 只在三类节点暂停**（方向选择/方案确认/质量验收），其余全自动。设计类 Agent 直接调 LLM API（快、便宜），Coder 类走 Claude Agent SDK 隔离容器（完整文件系统与工具链），设计产物经全局状态自动流入 Coder 系统提示词，无需"交接"环节。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

### 工程基建：壳应用"固化层 + 可变层"

每个 Coder 节点对接独立远程容器，预装壳应用。小游戏壳应用固化层：Phaser + React 脚手架、Controller 分层（业务/游戏/音频）、通用 UI 框架、状态管理、关卡查询推进与道具 CRUD 接口、资源加载与事件总线；可变层：游戏对象、主场景逻辑、UI 内容、关卡配置、道具效果。关卡生成壳应用固化层：Period → Day → Level 三级周期配置、通用服务接口、gameType 注册机制、Schema 驱动配置、构建发布流程；可变层：具体生成算法与验证逻辑。宿主-游戏通信协议三条通道：统一数据查询 `queryGameConfig`（一个通用异步函数承载所有请求）、标准生命周期单向回调、双向事件协议（`app:event` 上行 / `host:event` 下行）。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

## 运营 Agent：主持人 + 专家团对抗模式

单 Agent 做运营分析易把方案做得过简、归因到无法行动的客观因素、或给出缺证据的结论。因此采用"**主持人 + 专家团**"多 Agent 模式：每个专家先**独立看数**形成自己的证据链（避免过早锚定），不同任务有各自系统 Prompt（明确步骤、门禁和**已知死路**，开工时强制注入），专家围绕具体方案互相论证，分歧在对话中解决，最后由主持人收敛——不把未经处理的争论抛给用户。产出两类结果：现有游戏迭代提案、新游戏立项。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

### 无人值守四道门禁

每日定时任务（非对话式，系统以决策者身份自动判断"今天最值得改哪款游戏"）最需避免的是**把错误结果写入待评审队列和知识库**：

1. **数据整批验收**：核心报表构建失败，当天所有任务中止——宁可不跑，不让错误数据生成结论
2. **验收真实状态**：会话正常退出 ≠ 任务完成，系统检查立项书是否新建、提案是否更新、待结算清单是否清空；未通过时同一会话催促续跑，预算耗尽才标记失败交人工
3. **自动巡检**：每半小时回收假死任务、补跑漏批次、纠正"看起来完成、实际没有交付物"的状态
4. **统一交付标准**：自动化只省掉中途等待同意，查重/读代码/证据要求与交互式一致，产物进待评审队列，评审权在人^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

### 能力自进化：可证伪的"下注" + 四层记忆

Agent 交付物被重新定义为**一次可证伪的"下注"**：每个策略写明预测区间、引用了哪些知识、什么结果会证伪当前判断。AB 数据回收时逐条对账预期与实际，并追溯判断引用的知识——知识本身有问题就进入治理，而非只标记实验成败。四层记忆按不同成本/容量/检索方式分容器；记忆正文只保留"结论 + 判断依据"，不存一次性细节和易过期数值；治理元数据含作用域（单款/跨游戏）、来源证据（哪次结算哪个会话）、**证据等级**（现象/机制/未验证推断——低等级禁止"显著""必然"等强断言）、置信度与状态（候选/生效/失效）、版本链（回溯生成时系统状态）。^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

## 实测数据

- **生产侧**：两周上线 6 款小游戏（忍者跳跃/盖楼/飞机大战/贪吃蛇/打地鼠/祖玛）；最快一款 Agent 平台内走完生产流程约 1 小时，含测试验收上线从创意到上线仅 2 天；测试/PD/运营均可直接生产（生产关系改变）
- **运营侧**：8 月底自运营上线后共 20+ 次迭代和 1 款新游戏；首周 12 款游戏全面迭代，**95%+ 迭代正向、40%+ 全指标正向**；从方案产出到开发上线仅 3 天^[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23.md]

## 与相关实体的关系

- 与 [[entities/tarot-pixel-context-loop-engineering-visual-reduction-aliyun-2026|Tarot Pixel 上下文工程 + 循环工程]] 同属阿里系第一方 Agent 工程：Tarot Pixel 是单环节（视觉稿还原）的降噪+收敛方法论，本文是全链路（生产-运营-迭代）系统级落地，且生产系统的 Claude Agent SDK 代号 tarot-code-agent 与 Tarot Pixel 同源。
- 与 [[entities/aliyun-agentloop-enterprise-agent-self-evolution-flywheel|阿里云 AgentLoop 自进化飞轮]] 同为"自进化/自迭代"主题，但 AgentLoop 是平台产品视角，本文是具体业务（淘天互动小游戏）的完整工程实录，含 AB 实验对账与记忆治理细节。
- "预期收益-真实数据回收"的下注式对账与 [[entities/agent-evolution-four-stages-six-dimensions-aliyun|阿里 Agent 进化框架]] 的评估反馈思想同源，但本文落地到每日定时任务+门禁+证据等级治理的具体机制。
- 与 [[entities/hermes-agent-goal-runtime-architecture-state-persistence-judge-closed-loop|Hermes goal runtime 判定闭环]] 在"会话退出 ≠ 任务完成，需验收真实交付物"这一点上独立趋同。

→ [[raw/articles/minigame-self-iterating-agent-taobao-tian-2026-09-23|原文存档]]
