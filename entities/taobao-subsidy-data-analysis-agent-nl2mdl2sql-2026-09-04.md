---
title: "淘宝百亿补贴数据分析助手 Agent：NL2MDL2SQL + 多Agent协同 + 六层知识体系"
created: 2026-09-04
updated: 2026-09-07
type: entity
tags: [agent, data-agent, nl2sql, nl2mdl2sql, wrenai, semantic-layer, multi-agent, intent-routing, react, lineage, sql-lineage, metric-registry, attribution, evaluation, hologres, taobao, tmall, alibaba]
sources:
  - raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04
confidence: 0.78
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 淘宝百亿补贴数据分析助手 Agent：NL2MDL2SQL + 多Agent协同 + 六层知识体系

## 核心命题

淘天集团-天猫技术团队（伯略）为百亿补贴/聚划算这一核心营销阵地构建的数据分析智能体 `bybt-data-analysis-assistant`。用户用自然语言提问，Agent 自主完成从找表、写 SQL、执行查询到深度分析的全流程。系统最关键的选型是**摒弃直接 Text-to-SQL，改用 NL2MDL2SQL**——通过 WrenAI 的 MDL 语义建模层为 LLM 生成 SQL 确定性兜底，同时用多 Agent 协同（意图识别/调度路由/ReAct）和六层知识体系支撑深度归因。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

核心矛盾与解法：直接 Text-to-SQL 有**幻觉字段、口径错误、不可复用**三个致命问题——LLM 不知道 `ads_bybt_flow_sum_di` 表必须加 `activity_type='百亿补贴'` 这个 WHERE 条件，会编造字段、漏过滤条件、每次结果不一。NL2MDL2SQL 用 MDL 语义层让 Agent 生成 SQL 前先从表结构定义（ml/model.json）获取真实字段，使字段存在、口径正确、可复用三者都得到保障。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 四层架构全景

### 1. 意图识别层（Intent Agent）
Agent 的前哨——执行前先用**一次 LLM 调用**解析出标准化 JSON 意图参数包（意图分类+时间解析+指标识别+维度推断），如 `{"intent_type":"attribution","confidence":0.95,"metric_field":"gmv_amt","curr":"2026-06-14","prev":"2026-06-13","scope":"all"}`。降级策略：LLM 调用失败自动回退关键词匹配+规则解析。多意图检测：主意图放 `intent_type`，次要意图放 `secondary_intents` 数组。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### 2. 调度路由层（Supervisor）
Supervisor 根据意图识别结果分发到对应专家 Agent：Query取数 / Analysis分析 / Ops运维 / Other兜底。**不用单 Agent 的原因**：单 Agent 需要巨大 system prompt + 几十个工具，导致工具选择困难、prompt 膨胀、调用预算浪费；多 Agent 让每个专家只看自己相关的工具和指令。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### 3. 执行层（ReAct 推理循环）
Agent 边想边做，自主决定调什么工具、传什么参数、什么时候收敛。以「昨天 GMV 对比」为例走完整的 Thought→Action→Result 循环：找表→查规则确认口径→build_sql→odps_query→python_analyze 归因。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### 4. 保障层（安全守卫/Harness + SQL Validator Gate）
Agent 有「刹车系统」（Harness）防失控。所有生成的 SQL 经 **SQL Validator Gate** 后置校验（语法正确、表名存在、字段合法）。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## NL2MDL2SQL：核心技术选型

### WrenAI MDL 语义层
MDL（Model Definition Language）类似 dbt 语义建模层。四能力：**mdl.json**（模型定义文件：表名/字段/关系/描述）、**dry_plan**（NL→SQL 预演）、**recall**（查询历史召回）、**fetch_context**（上下文召回，给 LLM 补表结构信息）。生成的 SQL 经语义层校验确保字段存在、口径正确、可复用。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### Hologres 外部表加速
数据分析场景多次递归下钻，每次 ODPS 执行 30-120 秒，5-10 次 SQL 可超 10 分钟。引入 Hologres 外部表直读 ODPS，**单次查询加速到 3-10 秒**。选外部表而非全量同步：不需 DBA 介入、onboard 新表自动挂载、首次自动创建外部表、后续 LRU 缓存命中。**场景闸门**：仅对 `stream_analysis`（多步分析编排器）生效，普通 NL2SQL/CLI/运维查询仍走 ODPS。

执行链路（odps_query 接口签名不变）：变量替换 → 黑名单检测（GET_JSON_OBJECT/EXPLODE 不兼容函数直走 ODPS）→ sqlglot AST 抽表名 → 按需创建外部表（`IMPORTFOREIGNSCHEMA <project> LIMITTO (<table>)`，LRU缓存+持久化JSON避免重复DDL）→ sqlglot 方言转换（ODPS→PostgreSQL）+ 表名重写 → aistudio HTTP 网关执行 → 失败 fallback ODPS。**切库自愈**：检测 `relation does not exist` 错误→清 IMPORT 缓存→重新 ensure 外部表→重试一次，全程对 Agent 透明，最多增加 3 秒。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 六层知识体系

| 层 | 内容 | 权威 |
|----|------|------|
| L1 | 表能力清单 + MDL 语义模型 | |
| L2 | SQL 片段库 (Snippet Store) | |
| L3 | SQL 规则库 (Rule Store) | |
| L4 | 血缘知识图谱 (LightRAG KG) + AST 实时解析 | |
| L5 | 指标注册表 (Metric Registry) | 波动归因核心 |
| L6 | 会话记忆 (Session Memory) | |

Agent 在 ReAct 循环中主动调用知识检索工具，每层知识对应不同工具：search_metrics(L5) → cap_search(L1) → snippet_search(L2) → rule_search(L3) → lightrag_table_lineage(L4 KG) → parse_sql_lineage(L4 AST实时)。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### AST 血缘解析引擎
基于 sqlglot 的 SQL 静态分析管线，从 ETL 源代码自动提取字段级血缘。**六阶段**：①Schema Registry（加载字段定义，DataWorks API 降级）②预处理（全角标准化/DDL剥离/变量替换/多INSERT拆分）③AST标准化（展开 SELECT*）④节点提取（DFS 遍历，CTE/子查询/UNION分支/主查询 scope）⑤作用域解析（符号表+alias 消歧，CTE scope 隔离拓扑序解析）⑥血缘映射（direct/computed/aggregated 三类边）→输出标准化 JSON。**两种使用模式**：预先解析灌入 KG / 实时降级链（lightrag → dw_get_table_source 取 ETL 源码 → parse_sql_lineage AST实时解析）。回归测试覆盖 **45 张真实表、296 个节点**。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### 指标注册体系（Metric Registry）
波动归因诊断的核心知识源——定义每个业务指标如何拆解。指标定义成为**声明式配置**，运营同学通过 Web 向导即可接入新指标，而非靠 LLM 猜（LLM 可能把 GMV 拆成流量×客单价、漏掉转化率）。

MetricDef 以 GMV 为例：`formula_type=multiplicative`，factors=[IPVUV(column), CVR(ratio: ord_cnt/ipv_uv), AOV(ratio: pay_amt/ord_cnt)]。**乘法型恒等式铁律**：因子相乘后约分必须恰好等于 total_col——`GMV = ipv_uv × (ord_cnt/ipv_uv) × (pay_amt/ord_cnt) = pay_amt`。维度按两个正交轴分类（perspective/axis）。AI 辅助接入：选 MDL 表→AI 生成 MetricDef（自动修正 perspective/axis 混淆）→Pydantic 校验→预览恒等式→入库。公式注册表直接驱动归因诊断树：metrics.yaml → formula_registry（漏斗/加法/比率公式）→ 归因引擎多路筛选最显著拆解 + 递归下钻 → 诊断树报告。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 分析能力

### 波动归因诊断（Analysis Agent）
「昨天 GMV 为什么跌了 20%」→ 构建**诊断树**：GMV=访客数×转化率×客单价 → 转化率-22%主因 → 按行业拆家电-45% Top1 → 家电下单量-40%主因 → 结论「家电行业下单量骤降导致 GMV 下跌」。多路筛选（漏斗/加法/比率公式选最显著）→ 递归下钻沿主因子逐层拆。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

### 血缘溯源（KG 优先、AST 降级）
路径1：查 LightRAG 知识图谱 `lightrag_column_logic`。路径2：KG 未命中时 AST 实时解析 ETL 源码 `dw_get_table_source → parse_sql_lineage` 返回字段映射、来源表、WHERE 条件。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 评测体系（6D 意图感知）

评测回答 3 个问题：评什么（**6D 维度**）、怎么构造样例（**Golden Dataset**）、怎么打分（**意图感知权重 + 节点归因**）。

- **6D 评分**：每个维度 0-100 —— D1 factor / D2 tree（深度+分支数+方向）/ D3 tool（must_call 40+must_not_call 25+ordering 20+step 15）/ D4 conclusion（LLM-as-Judge 按意图 Rubric）/ D5 sql（语法/表名/字段/WHERE/白名单）/ D6 faithfulness（规则检查+LLM 幻觉检测）。
- **意图感知权重**：不同意图关注点不同——归因不关心 SQL 正确性（D5=0），取数不关心因子命中（D1=0）。每意图配置差异化权重，权重为 0 的维度显示 "--" 不参与打分，Web 界面可自定义。
- **Golden Dataset**：当前覆盖 6 类意图、15 用例。expected 多层校验（不只看最终回答，还看工具调用序列、SQL 内容、诊断树结构）；`main_factor_fuzzy` 模糊匹配兜底；每类意图≥2 用例覆盖错误输入；Web 评测中心在线增删。
- **评测执行流程**：harness.run_case → 收集 SSE 事件流 → EvalTrace（events/tool_calls/diagnosis_tree/final_answer/nodes）→ 6D 打分 → 加权总分 Σ(Di×Wi) → 报告输出（终端/JSON/Bad Case 归档 总分<60 或任一维度=0/版本对比 --compare）。
- **节点级归因诊断**：从事件流提取 7 个关键节点状态，自动定位失败根因，如 `gate - validation_exhausted(重试3次未通过)` / `tool_call - 工具返回错误 odps_query` / `quality - 各节点成功但质量未达预期`。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 新表接入（Onboard 五步自动化流水线）

①表结构同步（ODPS schema→mdl.json+TisPlus3）②表类型探查（A预聚合/B普通+主键+维度）③规则抽取（下游血缘代码→LLM 提取 WHERE 条件）④SQL 模板生成（B类表自动标准模板）⑤入库（规则/片段/能力清单写入 TisPlus3）。支持 Web 界面与 CLI 两种方式。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]

## 与已有实体的关系

- **`trust-infrastructure-ai-data-retrieval-taobao-2026`**（代构层+确定性 SQL 引擎，AI 不写 SQL）：mechanics 不同但同属淘天取数信任主题。本文 NL2MDL2SQL 是**全 LLM 自主生成 SQL + MDL 语义层兜底**，trust-infra 是**工程确定性生成 SQL、AI 只出查询请求**——两条互补路线，均解决 AI 写 SQL 幻觉，机制正交。^[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04.md]
- **`gaode-ai-native-data-agent`**（高德 NL2SQL + 规约约束 + 渐进式漏斗选表）：同为数据 Agent 生产实践，高德用规约约束，本文用 MDL 语义层 + 指标注册 + 归因诊断树，维度不同。
- **`agent-evaluation-fine-grained-system-aliexpress-2026`**（同品牌阿里，模块级白盒评测，质量×成本×性能）：本文 6D 评测是另一轴——**意图感知加权 + 节点级归因诊断 + Golden Dataset 多层校验**，与模块级白盒维度互补。

→ [[raw/articles/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04|原文存档]]