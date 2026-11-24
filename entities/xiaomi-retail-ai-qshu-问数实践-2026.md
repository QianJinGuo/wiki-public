---
title: "小米零售 AI 问数实践：从 Text-to-SQL 到 Text-to-Metrics"
created: 2026-08-18
updated: 2026-11-20
type: entity
tags: [ai, agent, text-to-sql, text-to-metrics, metrics-governance, rags, xiaomi, retail, semantic-routing, evaluation]
sources: [raw/articles/xiaomi-retail-ai-qshu-问数实践-2026]
confidence: 0.72
provenance_state: extracted
score: 72
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 小米零售 AI 问数实践：从 Text-to-SQL 到 Text-to-Metrics

## 核心洞察

一线店长每天要面对"店里库存压了多少""新品销售怎么样"等问题，看似只是"问个数"，但要让 AI 真正服务业务，还需解决**指标口径、查询规则、数据权限**等问题。小米中国区零售团队从 **Text-to-SQL 转向 Text-to-Metrics**，并围绕**指标治理、语义路由、查询执行、评测体系**进行工程化建设。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md]

## 关键设计

- **Text-to-Metrics 转向**：相比直接生成 SQL，先落到统一指标层，规避口径混乱与重复造轮子
- **指标治理**：统一指标定义、口径、语义，作为问答的稳定中间表示
- **语义路由**：把自然语言问题路由到对应的指标/指标族，而非逐问答建查询
- **查询执行**：指标语义到实际数据源查询的映射与执行链路
- **评测体系**：对问答准确率/口径一致性做系统化评估，形成闭环

## 与现有体系的关系

本文属 Agent 驱动数据访问 与 Agentic RAG 的具体工程案例，补充了 Text-to-Metrics 这一指标层范式。与 [[entities/multi-agent-architecture-retail-practice|零售多 Agent 架构实践]]、[[entities/xiaomi-retail-ai-engineering-three-layer-practice|小米 AI 工程化三层实践]] 同属小米/零售 AI 落地系列。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md]

## 深度分析

### 从"现场推理"到"离线治理"：确定性被工程化地前移

Text-to-SQL 路线的失败并非模型能力不足，而是把本应离线完成的口径判断压进了在线推理时刻：早期版本生成的 SQL 语法完全正确，却漏掉取消、退款、退货等过滤条件，结果比业务口径高出近 10%，背后是字段级歧义、状态过滤缺失、跨表关联三类无法从 schema 本身读出的业务约束。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:32-38] 引入 RAG 补足上下文后不确定性依旧，根源在于无论检索到多少文档，最终口径判断仍要靠模型当场完成；Text-to-Metrics 的本质正是把"选哪个指标"从事中搬到事前——口径、别名、过滤规则在治理阶段固化为机器可用的语义资产，模型只负责识别意图与填充参数。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:49-52] Text-to-SQL 路线本身也在另一方向演进（如 [[entities/revsql-task-expertise-rl-text-to-sql|用任务专长强化学习改进 Text-to-SQL]]），两条路线并无绝对优劣，小米以路由层统一调度：命中已治理指标走 Text-to-Metrics，未命中则降级回 Text-to-SQL。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:80-80] 同类"指标语义层先行"的范式也见于 [[entities/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions|Lyft 的指标语义层治理]] 与 [[entities/tencent-omega-ai-bi-marmos-2026|腾讯 Omega AI BI]]，可互为参照。

### RAG 的天花板：检索解决"找到"，解决不了"判定"

文章对 RAG 失败的拆解值得细读：碎片化上下文切断了指标之间的业务层级；"什么时候该用哪个字段"这类业务约定只存在于团队经验中，从未进入检索库；多个候选指标语义高度相似时，Top-K 排序无法替代最终的业务判断。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:56-62] 这划出了 [[concepts/retrieval-augmented-generation-rag|RAG]] 的收益边界：它擅长解决"知识缺失"型问题，而对"决策缺失"型问题（多个看似都对里选出业务认可的那个）必须引入确定性机制。"达成率"治理是最佳注脚——三个公式不同、字面描述高度相似的同名指标，靠决策表映射规则而非更多文档，把指标选择准确率从约 65% 提升到接近 98%。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:94-109]

### 纵深防御：把不确定性消灭在各自环节

工程链路呈现清晰的分层校验设计：语义路由按 8 个业务领域建立分区、加载上限设为 3 个分区，触发词命中过多时优先追问澄清而非扩大上下文；且 80% 以上真实查询只涉及 1-2 个分区，说明该上限来自查询分布数据而非主观设定。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:115-117] Alias 白名单保证最终执行的指标来自已注册列表，权限预检提前区分"指标不存在"与"用户无权限"，零结果保护把空结果视为有效业务结论、禁止系统换日期换条件反复重试这种掩盖问题的行为。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:123-127] 实体消歧在无法精排时主动请用户确认，数据访问范围按角色在查询构造阶段注入、完全不交给模型决定。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:136-140] 这正对应工程复盘中的"不确定性不向下游传递"：每个环节只解决自己能确定的部分，其余升级为追问或中止，而不是让模糊一路传播到最终查询。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:205-205]

### 双维评测：把"对不对"拆成两个可定位的变量

DSL 准确率与答案准确率分开评测是全文方法论价值最高的设计：维度少一级但答案通过其他方式补全正确时，两者分离恰好定位出问题出在查询构造而非最终结果。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:155-157] 时间口径 badcase 案例进一步验证：17 条失败中 8 条集中在同一根因（时间字段偏移处理），修复后 DSL 准确率不变仍为 93.2%，但新增 44 条专项用例的答案准确率首次达到 100%——单看 DSL 是否一致不足以判断系统最终答案是否正确。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:161-174] 离线侧从 BI 看板自动收录真值构造评测集、周期执行并在准确率波动时告警，让评测基准随业务持续更新而非一次性人工标注。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:178-184] 这套做法可与 [[concepts/evaluation-harness-design|评测体系设计]] 的通用模式对照。

## 实践启示

1. **按查询场景选路线，而非追新范式**：核心用户要可信结果、查询集中在固定高频场景、结果直接驱动经营决策时，指标层范式优于自由生成 SQL；两条路线由路由层统一调度、未命中即降级，保留灵活性。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:69-80]
2. **指标治理从决策表开始，而非从写文档开始**：对"达成率"这类多口径指标，价值在于把公式差异、适用对象沉淀成"用户表达 → 指标"的精确映射规则，无修饰的表达按用户归属自动判断。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:100-102]
3. **用加载上限约束上下文膨胀**：语义分区 + 上限 3 个分区 + 超限追问的组合，比"塞更多文档"更能控制检索噪声，且上限由真实查询分布数据支撑。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:115-117]
4. **权限与安全逻辑永不交给模型**：数据访问范围在查询构造阶段按角色自动注入，跨门店对比等越权意图走独立的三重判断分支，即使上层模型产生偏差也有兜底。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:138-140]
5. **评测集要随 badcase 生长**：修复专项问题后新增专项用例做回归验证，而非沿用原测试集；每条失败都进入"发现问题 → 分析原因 → 修复优化 → 回归验证"闭环。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:172-174]
6. **以真值自动化评测替代人工抽检**：从 BI 看板自动收录真值、按线上真实 Query 模式构造评测集、周期执行并在准确率波动时自动告警，覆盖日常迭代引入的回归问题。^[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026.md:178-184]

## 价值

- 提供了从 generator 直接写 SQL 到"先落指标层再查询"的**可迁移工程范式**，对数据问答类 Agent（BI、经营分析、智能客服）有直接参考价值
- 指标治理 + 语义路由 + 评测闭环的完整链路值得复用

→ [[raw/articles/xiaomi-retail-ai-qshu-问数实践-2026|原文存档]]
