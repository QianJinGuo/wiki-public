---

title: "京东健康OPC团队的产品全流程Skill探索"
created: 2026-07-05
updated: 2026-11-08
type: entity
tags: [ai, agent, llm]
sources: [raw/articles/京东健康opc团队的产品全流程skill探索]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 京东健康OPC团队的产品全流程Skill探索

## 京东健康OPC团队的产品全流程Skill探索

^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

## 京东健康OPC团队的产品全流程Skill探索


一、背景和适用场景

  


京东健康正在探索OPC（One Person Company）下模式的高效交付，在 OPC 团队中，没有专职产品角色，这意味着团队不仅要关注「把需求做完」，还必须回答几个更前置的问题： ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

  * 这个问题是否真的值得做？

  * 证据来自用户、数据、业务，还是只是个人判断？

  * 当前方案是不是唯一方案？

  * 需求边界是否清楚？

  * 当前迭代是否真的有产研资源承接？

  * 上线后如何判断成功或失败？

  * 风险、进展和结果如何对不同对象同步？

Anthropic 开源的 Product Management Skills （https://github.com/anthropics/knowledge-work-plugins/tree/main/product-management/skills）正好可以承接这些问题。它本质上是一套**产研共同决策框架** ：把从发现问题到上线复盘的过程，拆成一组可重复使用的动作，帮助 OPC 在缺少专职产品角色的情况下，仍然保持清晰、可追溯、可评审的产品流程。 ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

本文把这套实践适配到 OPC 的工作场景，快速补齐OPC团队的产品能力，跑通从需求发现、定义、排期、开发到上线复盘的全过程。 ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]


二、流程总览

  


###  2.1 安装 Skill

这套实践在 Anthropic 原生环境中对应 product-management plugin；在 JoyCode 或 Codex 中，需要接入 `product-management/skills` 下的 8 个 Skill。在对话框输入以下提示词即可安装： ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

  *   *   *   * 

从 https://github.com/anthropics/knowledge-work-plugins/tree/main/product-management/skills安装以下 skills：synthesize-research、metrics-review、competitive-brief、product-brainstorming、write-spec、roadmap-update、sprint-planning、stakeholder-update。 ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

### 2.2 一条判断链

8 个 Skill 最常见的使用顺序如下：^[raw/articles/京东健康opc团队的产品全流程skill探索.md]


这条链的内在逻辑是：

  1. **先判断问题是否值得做** （问题判断）；

  2. **再把方向打磨成可执行需求** （需求定义）；

  3. **再把需求放进优先级、容量和依赖约束** （排期执行）；

  4. **最后用指标验证结果，并完成同步** （上线复盘）。

可以把它当成一套轻量工作协议：**任何需求只要进入开发，就至少要经过「证据、假设、边界、排期、验收、复盘」六个环节。** ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

****


三、第一阶段：问题判断

  


> **本阶段目标** ：判断「这个问题是否足够重要」，而不是证明「我们想做的方案是对的」。

需求刚出现时，不要急着写 PRD，也不要直接进入技术方案。第一步是判断问题本身是否成立。^[raw/articles/京东健康opc团队的产品全流程skill探索.md]


### 3.1 使用哪些 Skill

### 3.2 怎么接入需求上下文

Anthropic 的做法不是让团队手工整理一份完整报告后再交给 AI，而是把现有材料按来源接入到 Skill 工作流里。以 synthesize-research 为例，它支持三类输入： ^[raw/articles/京东健康opc团队的产品全流程skill探索.md]

  * **直接粘贴** ：访谈记录、会议纪要、问卷开放题、工单摘要。

  * **上传文件** ：研究文档、表格、录音转写摘要、调研结果。

  * **连接工具** ：从知识库、用户反馈系统、产品分析工具、会议转写工具中拉取上下文。

在 Anthropic 的插件设计里，这些工具用通用占位符表示：^[raw/articles/京东健康opc团队的产品全流程skill探索.md]


### 我们无法直接使用这些海外工具的连接器，但思路完全可以复用：**把内部系统的材料映射到同样的证据类别** 。

### 关键是**不要把材料无差别丢进去** 。每份材料进入 Skill 前，至少要带上 5 个字段：

  *   *   *   *   *   * 

  

-> [[raw/articles/京东健康opc团队的产品全流程skill探索|原文存档]]^[raw/articles/京东健康opc团队的产品全流程skill探索.md]


---
## 深度分析

1. **Skill 链的本质是把产品判断外置成协议，而不是工具堆叠。** 8 个 Skill 串成「问题判断 → 需求定义 → 排期执行 → 上线复盘」四阶段，其真正的产出物不是 PRD 或计划表，而是一条强制走完「证据、假设、边界、排期、验收、复盘」六个环节的轻量工作协议。对没有专职产品角色的 OPC 团队而言，这相当于把资深产品经理的决策框架编码成了可复用的工作流——与 [[concepts/skill-engineering-principles]] 中「skill 承载流程知识而非工具调用」的思路一致。

2. **证据接入层是这套方案里最容易被低估的设计。** 文章强调材料进入 Skill 前必须带「来源、时间、对象、内容、关联」5 个字段，本质是给 LLM 的输入做了 schema 化：没有出处和时间戳的材料产出的结论置信度低，这把传统研究方法里的证据分级实践迁移到了 AI 工作流中。海外工具连接器虽不可用，但「把内部系统材料映射到证据类别」这一抽象层是可移植的——工具会变，证据 schema 不会。

3. **问题判断与方案验证被刻意分离，防止 Skill 沦为确认偏误的放大器。** 文章反复警示「不要用 Skill 来证明自己想做的方案是对的」，第一阶段的目标被严格限定为判断问题本身是否成立。这是对 AI 辅助决策的一个关键认知：LLM 擅长为给定方向找论据，若工作流设计不预先卡住问题定义环节，skill 化反而会加速错误方向的执行。

4. **容量约束和取舍说明是 resource-reallocation 思维的体现。** 「新增需求不是多做一次，而是一次资源再分配」以及「只规划 70%-80% 容量」这两条规则，把敏捷开发中的容量管理硬编码进了 roadmap-update / sprint-planning 的工作流。每份 Roadmap 变更说明必须写清取舍结果，这使得排期决策从口头博弈变成可审计的记录。

5. **对 OPC 场景的适配揭示了 skill 落地的真实成本结构。** 文章给出的最小启动路径（先只用 synthesize-research / write-spec / metrics-review 三个 Skill 跑通一个真实需求，再补齐其余环节）说明：8 个 Skill 一次性铺开不如先用最小闭环建立团队信任，模板沉淀（材料输入格式、需求规格、变更说明、复盘清单）才是长期资产。这与 [[entities/jd-health-opc-skill-full-process]] 记录的孪生实践可互相印证。

6. **Skill 编排顺序本身就是一种隐性知识管理。** 8 个 Skill 单看都是通用动作（研究综合、竞品简报、头脑风暴、写规格、路线图、排期、复盘、干系人同步），它们在 Anthropic 的产品方法论里各有位置；京东健康 OPC 团队的贡献在于把「先发散后收敛、先定位后落计划、先指标后同步」这条使用顺序显式化并写成启动提示词。这与 [[concepts/skill-framework-writing-patterns]] 讨论的「skill 的价值一半在内容、一半在编排」相互呼应——同一组 Skill，编排错了（例如跳过问题判断直接 write-spec）就会退化成加速错误决策的流水线。

## 实践启示

1. **用 Skill 链补齐角色缺口，而非替换角色。** 没有 PM 的团队可以先用 Anthropic 的 8 个产品 Skill 把「问题判断 → 需求定义 → 排期 → 复盘」跑起来，但 Skill 产出仍需产研共同评审——工具承接的是结构，判断仍归团队。

2. **给进入 AI 的材料定 schema。** 任何调研/工单/数据进入 Skill 前，先补齐「来源、时间、对象、内容、关联假设」五个字段；缺字段的材料宁可不用。这是低成本提升 AI 分析置信度最直接的手段。

3. **把六个环节设为进入开发的门槛。** 「证据、假设、边界、排期、验收、复盘」可以直接作为团队的 DoR/DoD 检查项：缺证据不进定义、缺验收标准不进排期、上线不复盘不结项。

4. **排期写死两条纪律。** 只规划 70%-80% 的产研容量，剩余留给线上问题与返工；每次新增需求必须附带 Roadmap 变更说明（变更内容、原因、影响、取舍结果、风险依赖），让「做这个 = 不做那个」显性化。

5. **从三 Skill 最小闭环起步，再固化模板。** 新团队试点先用 synthesize-research（问题判断）、write-spec（需求规格）、metrics-review（上线复盘）跑一个迭代大小的真实需求，跑通后把形成的模板固化为团队文档复用——流程的持久价值在模板资产，不在 Skill 本身。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构
- 同题异语种孪生页：[[entities/jd-health-opc-skill-full-process]]（归并候选，提案卡 #11 批1）


