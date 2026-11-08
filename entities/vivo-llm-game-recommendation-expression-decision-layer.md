---
title: "vivo LLM 游戏推荐表达层：从\"推什么\"到\"怎么选\""
created: 2026-07-08
updated: 2026-11-09
type: entity
tags: [llm-application, recommendation-system, game-distribution, prompt-engineering, llm-harness, structured-output, vivo]
sources: [raw/articles/vivo-llm-game-recommendation-expression-decision-layer]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# vivo LLM 游戏推荐表达层：从"推什么"到"怎么选"

> vivo 互联网产品团队在生产级游戏分发场景中，用 LLM 补上推荐系统的"最后一公里"——不改排序，通过 LLM 理解游戏、表达差异，帮用户走完从"给结果"到"帮决策"的最后一步。^[raw/articles/vivo-llm-game-recommendation-expression-decision-layer.md]

## 背景

推荐系统擅长回答"推什么"，用户却卡在"怎么选"。尤其游戏场景决策成本高（下载试玩需投入时间、可能的付费、社交关系），用户需要先看懂差异才愿意尝试。排序的边际收益递减，真正没被接住的是"看懂差异、把选择走完"。^[raw/articles/vivo-llm-game-recommendation-expression-decision-layer.md]

## 方法：探索→收敛双阶段

**第一阶段（模型探索）**：用大模型见多识广的优势自由拆解游戏——玩家为什么持续玩、爽感从哪来、成长循环怎么转、付费被什么驱动。这一步要的不是标准答案，而是尽可能多的"惊喜"和"边界"。^[raw/articles/vivo-llm-game-recommendation-expression-decision-layer.md]

**第二阶段（人工收敛）**：把探索里反复出现、真正能解释差异的维度挑出来，定成闭集 schema，再让模型在体系内做填空。**仅保留同时满足可解释、可比较、可复用、有决策价值的维度。**

> 自由生成是大模型的能力上限，稳定输出得靠约束。

## 核心架构：品类配置三层解耦

```
品类配置 = core_dimensions + expression_schema + highlight_priority
```

| 模块 | 职责 | 设计原则 |
|------|------|---------|
| **core_dimensions** | 比什么 | 跨品类对齐比较维度（成长驱动、刷宝驱动、爽感刺激……） |
| **expression_schema** | 怎么说 | 预定义表达模板，把描述格式钉死 |
| **highlight_priority** | 谁上卡片 | 按品类玩家最先想知道什么人工排序 |

新品类原则上加一行配置、不动代码。^[raw/articles/vivo-llm-game-recommendation-expression-decision-layer.md]

## 五条可复用工程经验

1. **探索和生产用模型的两副面孔** — 先发散探索再收敛执行，不可混用
2. **模型的原始输出不算接口** — normalize 层（闭集 schema + 缺字段回落 + 脏值过滤）是进生产的前提
3. **Prompt 的结构决定理解的深度** — 喂给模型的结构就是它输出的上界
4. **把"比什么""怎么说""谁上卡片"解耦** — 三件事塞一个 prompt 一把出，跨品类必飘
5. **让能力不绑上下文才能复用** — 无状态、不绑定调用场景的 LLM 插件

^[raw/articles/vivo-llm-game-recommendation-expression-decision-layer.md]

## 与现有知识体系的关系

- **normalize 层是 Harness 工程的具体实现** — 与 腾讯 Token 优化 的"约束金字塔"理念一致：模型的原始输出不做接口，外层确定性结构才是接口
- **探索→收敛的工作流** 与 improving-agents-data-mining-perspective-langchain 的 Agent 迭代思路相通：先发散再收敛，先探索再约束
- **AI 把"系统结构"变成产品的一部分** — 印证了 [[entities/agent-vs-workflow-control-continuum-framework]] 中控制权漂移的趋势：工程实现与产品设计的边界正在模糊
- **LLM normalize 层** 对应 Harness 工程中"评估器"的工程化落地——非确定性 LLM 输出的结构化封装

## 局限

- 当前产出仍属验证阶段，部分场景正在接入但未全量上线
- 品类配置依赖人工维护（但已做到"加一行配置、不动代码"）
- 准确性的保障依赖 normalize 兜底而非模型自身可靠性

---

→ [[raw/articles/vivo-llm-game-recommendation-expression-decision-layer|原文存档]]

---

## 深度分析

### 探索→收敛双阶段为什么有效

这套流程建立在对 LLM 能力边界的精确认知上：**自由生成是模型的能力上限，稳定输出是工程的下限要求**，两个阶段分别兑现这两件事。

- **探索阶段押注模型的"见多识广"**。玩家为什么持续玩、爽感从哪来、付费被什么驱动——这些维度不是团队能凭空设定的。旧标签体系的失败恰恰证明了这一点：它能说出"捕鱼类·PVE·模拟·轻度松弛"，却回答不了"为什么玩、为什么付费"。真正有决策价值的维度（游戏动机、付费机制）是模型在自由探索中"撞"出来的，不是先验规划出来的。
- **收敛阶段用人的判断立规矩**。从探索产物中只保留同时满足"可解释、可比较、可复用、有决策价值"的维度——这四个过滤器全是人类判断，模型不参与收敛。分工因此清晰：发散交给模型，收敛和立规矩由人来做。
- **顺序反了就全错**。如果"先定体系、再让 AI 执行"，体系的天花板就是设计者的想象力；让模型先撞边界，体系的天花板才是模型的知识面。

一个反直觉的佐证：多游对比最初"直接把三款丢给模型讲差异"的轻量做法反而够用，而顺着"做正经系统"拆多步、加 pipeline、分段 prompt 之后效果反而退化。教训是**复杂化本身会吃掉价值**——只有当复杂版本产出不比直出版本少、还能稳定复现，复杂化才算进步。

### 品类配置三层解耦的工程权衡

`core_dimensions + expression_schema + highlight_priority` 表面是工程拆分，实质是把**三种不同变更频率的关注点**分离：

| 解耦出的问题 | 变更来源 | 不解耦的后果 |
|---|---|---|
| 比什么（core_dimensions） | 品类认知，随理解深化而变 | 维度调整污染表达模板 |
| 怎么说（expression_schema） | 表达规范，要求跨品类稳定 | 格式散落在 prompt 各处，无法审计 |
| 谁上卡片（highlight_priority） | 产品决策，随运营策略而变 | 改展示优先级要动理解逻辑 |

三件事塞进一个 prompt"一把出"，模型要在单次调用里同时完成理解、表达、排版三种约束切换，跨品类必飘。拆开后各自只需满足单一职责，新品类原则上**加一行配置、不动代码**。

代价同样明确：三层配置全靠人工维护（数十个品类草案、人工排序 highlight_priority）。这是用"配置维护成本"换"代码变更成本"的经典权衡——配置错了可以快速回滚，代码错了要走完整发布周期。更微妙的是，这些配置既是工程实现又直接决定用户看到什么：它们是长在系统语义层里的产品设计，印证了 [[concepts/harness-as-product-surface|Harness 即产品界面]] 的判断——AI 时代"系统结构"本身成了产品的一部分。

### LLM 做表达层决策 vs 传统排序模型的边界

vivo 的方案在推荐链路里划了一条清晰的红线：**排序不动，表达层只读不回写**。这条边界的依据值得拆开看：

- **职责边界**：排序模型回答"推什么"（谁最可能被点击），LLM 回答"怎么选"（这几个有什么区别）。排序的边际收益已递减，接不住的是用户面对多款相似候选时的犹豫——这是表达问题，不是排序问题。
- **能力边界**：旧标签体系擅长回答"这是什么"，LLM 的世界知识补上"为什么玩、为什么付费"这一层动机理解。但 LLM 不做候选召回、不做 CTR 预估——准确率责任仍在排序侧。
- **可靠性边界**：准确性保障不依赖模型自身可靠，而依赖 normalize 兜底。模型即使出错，输出仍是合法结构，只是内容质量下降——失败模式可控。

与"用 LLM 重做推荐"的路线相比，这个定位更保守也更便宜。Netflix 的 [[entities/genrec-towards-llm-native-recommendation-at-netflix|GenRec]] 走 LLM-native 生成式推荐（LLM 直接参与召回与排序），快手 [[entities/onereason-kuaishou-reasoning-recommender-system|OneReason]] 把推理能力引入推荐模型本体；vivo 则刻意停在表达层，让 LLM 只做它最有比较优势的事——理解与表达差异。选游戏场景并非偶然：下载试玩要投入时间、可能有付费、牵扯社交关系，**决策成本越高，"看懂差异"对转化的杠杆越大**，表达层就越值得单独建设。

### structured-output harness 的作用

normalize 层是整套方案里让 LLM 输出进生产的唯一门票，它体现的是 Harness Engineering 的核心命题：**模型的原始输出不算接口，外层的确定性结构才是接口**。具体由三件事构成：

1. **闭集 schema**：expression_schema 把描述格式钉死，模型做填空而非开放问答——输出空间从无限自然语言压缩到有限枚举，可比较性由此而来。
2. **缺字段回落**：模型漏填时用预设默认值兜底，下游永远拿到完整结构。
3. **脏值过滤**：超出枚举、格式错误的输出被拦在进库之前。

这层 harness 还有一个常被忽略的作用：**prompt 的结构决定理解的深度**。喂给模型的维度框架（成长驱动、刷宝驱动、爽感刺激……）就是它输出的上界——既是约束也是脚手架，与 [[concepts/prompt-engineering-fundamentals|Prompt Engineering 基础]] 中"结构化提示引导结构化思考"的原理一致。最后，这套能力被做成无状态、不绑定调用场景的插件——能力与上下文解绑，跨场景复用才有可能。

## 实践启示

1. **给探索自由，给生产镣铐** — 同一个模型要用两副面孔：探索阶段放开问，生产阶段闭集填空。两者混用要么得不到惊喜，要么拿不到稳定。
2. **先直出，再考虑复杂化** — 加 pipeline 不是天然进步。只有当复杂版本产出不比直出少、且能稳定复现时，才允许引入复杂度。
3. **原始 LLM 输出永远隔一层 normalize** — 闭集 schema + 缺字段回落 + 脏值过滤是进生产的最低配置；没有这层，LLM 功能只能停留在 demo。
4. **把"比什么/怎么说/谁上卡片"拆成独立配置** — 变更频率不同的关注点不要绑死在一个 prompt 里；拆开后新品类加一行配置即可，不动代码。
5. **给 LLM 找准链路位置，而不是重做推荐** — 不动排序、只读不回写，让排序继续答"推什么"，LLM 补"怎么说清楚"。决策成本高的品类（游戏、大额消费）优先建表达层。
6. **动机维度比属性维度更有决策价值** — 旧标签答"这是什么"，对推荐决策几乎没帮助；补上"为什么玩、为什么付费"才能解释差异、支撑比较。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
