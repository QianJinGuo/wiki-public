---
title: "vivo LLM 游戏推荐表达层：从\"推什么\"到\"怎么选\""
created: 2026-07-08
updated: 2026-09-07
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
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

