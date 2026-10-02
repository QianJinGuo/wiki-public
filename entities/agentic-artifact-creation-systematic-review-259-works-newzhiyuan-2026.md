---
title: "Agentic Artifact Creation 系统综述：AI创作如何从「会生成」走到「能交付」"
created: 2026-09-09
updated: 2026-10-02
type: entity
tags: [agent, agentic-creation, artifact, survey, verification, multi-agent, harness, evaluation, harness-engineering]
sources: [raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026]
confidence: 0.75
provenance_state: extracted
---

# Agentic Artifact Creation 系统综述：AI创作如何从「会生成」走到「能交付」

> 香港科技大学（广州）、浙江大学、中科大、清华、香港大学、悉尼大学、中山大学团队综述（arXiv:2608.28122），覆盖截至 2026-08-20 的 **259 项工作 = 230 个系统 + 29 个基准**。核心命题：传统一步式生成（接收指令→生成结果→结束）只适合小且易检查的任务；完整交付物（长文/网页/视频/软件）需要**持续构造（Agentic Artifact Creation）**——系统维护一个不断变化的可交付物，根据中间结果决定下一步生成什么、修改哪里、何时停止。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 功能架构（三个功能角色）

论文把构造过程抽象为三个持续循环配合的角色：^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

- **作品表示（Operational Representation）**——保存作品当前状态，决定系统能改到多细。可以是文档、场景图、可执行代码，也可以是带图层/节点/依赖关系的结构化表示。系统若只能看到最终截图，问题难定位；可定位结构（文本块、页面组件、场景对象）才能支撑局部修复。
- **构造策略（Construction Policy）**——决定下一步动作：结合任务要求、当前状态、已得反馈，选择继续生成、修改某处、调用工具、请求人工判断，或在证据足够时停止。
- **运行时验证（Runtime Verification）**——观察行动结果：文本核对引用、页面渲染点击、程序编译运行测试、音视频播放。检查结果直接进入下一步决策。

一次修复循环：明确完成标准 → 保留可定位状态 → 验证当前结果 → 诊断问题 → 策略选择动作 → 局部修改 → 重新验证。三个角色配合支撑三种能力——组合性（Composability）、可追溯性（Traceability）、可修订性（Revisability）。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 六类作品与构造难度

以最终交付物为中心分为：文本、二维视觉、音频、视频、空间作品、行为型作品（软件/网站/游戏/模拟）。文本与视觉可静态检查、长距离一致性难；音视频沿时间序列检查；空间作品维护几何/语义部件/物理约束；行为型作品取决于代码在交互中如何响应。构造难度由三维变量决定：决策间关联程度、错误何时可见、能否在保留已有成果前提下局部修复。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 多 Agent 协作的协调成本

拆子任务可降单步难度但增协调成本：共享状态、依赖关系、完成标准。拆得越细，协调/冲突处理/重新组装成本越高。共享的作品表示把各 Agent 分工落到同一个可检查可修改的交付物上，维持局部结果一致。短小易验证任务，一个能稳定调用工具的单 Agent 往往更合适。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 评估三层对象（总分无法指导修复）

- **最终作品**：要求是否满足、内容一致、真实场景能否完成用途。
- **构造轨迹**：错误发现早不早、修复是否局部、已通过部分是否保留、耗时/调用/监督成本。
- **Agent 系统本身**：同任务重复是否稳定、换起点是否失灵、用户能否改目标并保留控制、更新后是否回归。

诊断粒度必须和编辑粒度对得上；增加 LLM Judge 数量不保证证据独立（生成器与评价模型同家族共享知识缺口）。来源对照/结构化状态/渲染结果/运行行为/构造历史/用户结果是不同证据渠道；规则检查/专用模型/LLM Judge/人工评审是不同评价者。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 四项设计原则与六个未来方向

**设计原则**：(1) 把必须保留的要求写进作品状态并与具体页面/段落/镜头/测试建立联系；(2) 划清控制边界——执行能力和决策权限是两回事，置信度不能代替权限；(3) 让反馈导向修改——反馈需连接证据、诊断、可执行动作；(4) 修改后重新验证受影响状态——旧证据会过期，需按影响范围选择性重查。

**未来方向**：全局一致性、精准修复（检测→定位→修复是三种能力）、系统自进化（跨任务留存的观念需独立验证+版本管理+回滚）、持续个性化、人类决策权限落地到具体行动、开放式成果可靠评估（承认一个任务可有多个好答案）。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

## 深度分析

### 评估的真正短板：总分不携带修复信息

综述最锋利的判断是：把评估收束为一个总分，只适合比较结果，不适合指导修复——总分不说明问题在哪里、由什么引起，也不给下一步可执行的修改^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:95-97]。更隐蔽的陷阱是粒度错配：检查器发现整体叙事不连贯，系统却只能重写全文；评估器发现视觉层级有问题，编辑接口却只能重新生成整张图。反馈再准，落不进编辑粒度就等于没有反馈^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:99]。这条判断对当前以 benchmark 分数为导向的 agent 评测是一记提醒：分数排行榜回答不了「下一个 patch 改哪里」。

另一个常被忽略的证据问题是**评价者独立性**：当生成器与 LLM Judge 来自相近的模型家族，它们共享知识缺口、审美偏好和判断盲区，增加 Judge 数量并不能制造证据独立性^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:111]。综述给出的解法是把「证据渠道」（来源对照、结构化状态、渲染结果、运行行为、构造历史、用户结果）与「评价者」（规则检查、专用模型、LLM Judge、人工评审）拆成两个正交维度，一次评价信号是两者的组合^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:113]——这与 [[concepts/evaluation-harness-design|Evaluation Harness]] 中多渠道证据回灌的设计同构。

### 构造难度是一个三维变量，而非按媒介排名

六类作品不构成难度排行榜。真正的难度由三个变量决定：决策之间的关联程度、错误何时可见、能否在保留已有成果的前提下局部修复^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:83]。文本与视觉可以静态检查、容易定位，但长距离一致性是硬骨头——改前文一个事实，后文的论证、图表、引用要连坐更新^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:77]；音视频的错误要到播放/渲染后才暴露^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:79]；行为型作品（软件/游戏）执行路径越长，获得有代表性观察的成本越高，失败越难追溯到具体状态^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:81]。用这个三维框架看，任何新系统都可以先问三个问题再谈架构：错误多早能被看见？修一处要牵动多少处？已通过的部分能否免于重做？

### 多 Agent 不是免费午餐：协调成本是隐藏税

拆子任务降低单步难度，却引入三类协调成本：共享状态、依赖关系、完成标准^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:87-91]。综述用一个非常具体的场景点破：文案 Agent 把文字改长，页面 Agent 是否知道版面已失效？事实核查更新了一个数字，图表与结论是否同步？每个 Agent 局部都完成了，全局结果仍可能互相冲突^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:89]。解法不是更多沟通，而是**共享的作品表示**——把分工落到同一个可检查、可修改的交付物上^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:93]。对短小易验证的任务，一个能稳定调用工具的单 Agent 往往更合适；多 Agent 的辩护责任在于证明中间观察确实改变了后续行动。

### 设计原则的公共内核：要求进状态，权限划边界

四项设计原则表面分散，内核一致。其一，把必须保留的要求写进作品状态并与具体页面/段落/镜头/测试挂钩，使「一次修改影响了哪些承诺」变成可查问题^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:117]。其二，控制边界：执行能力与决策权限是两回事，置信度不能代替权限，创意取舍、事实确认、对外发布需要不同授权级别^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:119]。其三，反馈要连接证据、诊断和可执行动作^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:121]。其四，修改后按影响范围选择性重验——旧证据会随版本过期^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:123]。综述同时诚实指出控制不是免费的：会推高表示、验证、协调成本，采用构造循环前需确认中间观察真能改善决策^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md:125]。

### 同构观察：这是 Harness 工程在创作域的完整叙事

把三功能角色与 [[entities/agent-harness-engineering-survey-2026|Agent Harness 工程综述]] 的工具+控制逻辑对读，可以看出两条独立收敛出的同一结论：可定位状态决定可修复性，验证证据决定策略质量。本综述的独特贡献是把这条线推到了「交付物」这个终点——Harness 讨论的是 agent 如何被约束，本综述讨论的是**被约束的 agent 最终如何交付一件持续变化的作品**。六个未来方向中，「检测→定位→修复是三种能力」（精准修复）与「跨任务留存需独立验证+版本管理+回滚」（系统自进化）尤其值得与 [[concepts/harness-engineering-framework|Harness Engineering]] 的演化议题合并跟踪。

## 与 wiki 焦点框架的连接

这是对 [[concepts/harness-engineering-framework|Harness Engineering]] 在**内容生成交付域**的系统化映射：作品表示 ≈ Harness 的 state；构造策略 ≈ 控制流/工具调用；运行时验证 ≈ [[concepts/evaluation-harness-design|Evaluation Harness]] 的证据回灌。与 [[entities/agent-harness-engineering-survey-2026|Agent Harness 工程综述]] 互补——前者聚焦工具+控制逻辑，本综述聚焦**持续可交付物闭环**。其「总分无法指导修复」「评价三层对象」「收益区分『多算了算』vs『真更聪明』」与 [[raw/articles/rethinking-harness-evolution-evaluation-mozhi-space-2026|Rethinking Harness Evolution]] 的 test-time-scaling 对照同源演进。^[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026.md]

→ [[raw/articles/agentic-artifact-creation-systematic-review-259-works-newzhiyuan-2026|原文存档]]