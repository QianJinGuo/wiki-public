---
title: "淘宝AI购物助手在线评测落地实践：LLM-as-a-Judge 的日级工业化"
created: 2026-10-13
type: entity
tags: [agent-evaluation, llm-as-a-judge, online-evaluation, rubric, data-cleaning, taobao, alibaba, agent-quality]
value: 8
confidence: 0.85
sources: [raw/articles/taobao-shopping-assistant-online-eval-2026-10-12]
provenance_state: extracted
---

# 淘宝AI购物助手在线评测：LLM-as-a-Judge 日级工业化

大淘宝技术（★★★★★ 第一方）购物助手购后场景的**在线评测**完整工程实录：日均百万条会话、多域异构的线上 Agent 场景中把 LLM-as-a-Judge 做到日级全量工业化运行——与美团图灵评测（[[entities/meituan-turing-agent-evaluation-methodology-2026-08-06|离线分层评测方法论]]）互补，本文的不可替代维度是**线上脏数据清洗 + 分布式推理工程 + 日级全量的持续收敛闭环**。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## 问题定义：为什么离线评测和 AB 实验都不够

离线评测测构造好的数据（query 精心设计/工具返回可控），线上用户表达模糊带情绪、工具返回缺失异常、分布持续漂移——离线通过率达标≠线上体验好；AB 实验粒度太粗（转化率下跌无法定位是路由错了还是回复编造）。**执行成功率 = 路由准确率 × 回复正确率**（三层漏斗复合指标），目标是把质量观测从月度/季度人工抽检变成日级自动观测。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## 四层数据清洗（学界假设干净输入，本文核心增量）

线上原始日志不是为评测设计的，四层清洗 pipeline 随各域推进依次沉淀：^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]
1. **修复数据格式不一致**：各 skill 卡片协议独立、字段各异，逐域对标离线 Prompt 的数据定义修复解析+字段映射；卡片文本数千字拼接超长需截断。
2. **语义对齐**：用户说"已揽收"vs 系统英文枚举值不天然等价更≠"已发货"——Prompt 中显式建立用户语义与系统语义映射，否则 Judge 按字面误判。
3. **屏蔽干扰信号**：tool_calls_json 含工具接口内部生产链路提示词，会被 Judge 误读为 Agent 回复——正则剔除只保留调用参数与返回值，屏蔽后各域准确率均明显提升。
4. **过滤规则精化**：入口过滤过宽导致大量非本域 query 混入（咨询政策≠真实发起操作），按 skill 标识精确过滤——某流程域通过率异常偏低的真因是 Prompt 不适配线上数据而非 Agent 质量差。

## 分布式推理工程：数仓与推理平台解耦

"数据归数据、推理归推理"：推理平台只做 LLM 推理，清洗/拼接/过滤与解析/归一化/聚合全在数仓 SQL——改 Prompt 不动数据逻辑，改过滤不重跑推理。按域拆独立 workflow（避免短板效应/故障扩散/迭代耦合），单域数千到数万条/天、运行 2-3h。工程踩坑：**Check 节点校验 input/output 条数一致**（公共资源池高峰超时丢数；回刷分区是叠加非覆盖，不清空会重复计算）、**token 预检**（超上下文窗口平台静默返回空不报错）、错峰调度+关键域独占资源。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## CNT 六层漏斗 Rubric 框架（归因基石）

全域统一的六层漏斗，层间短路串联（任一层判 0→后续层 N/A、整体判 0），**RED 恒评不受短路影响**：^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]
- **CTX 上下文门控**：证据不足整条标不可评不进分母——"诚实跳过优于强行评分"（唯一产生 N/A 的层）
- **OOSR 越界拒答**：query 不该由当前 skill 承接时是否正确拒答
- **OOC 能力边界诚实**：范围内但超能力时是否诚实说明而非编造
- **TCC 工具执行链路**：工具/参数/链路自洽
- **CNT 内容文本**：正确性/完整性/表达，域差异最大层（时效域防凭空"已签收"、检索域逐条校验时间/商品/属性/状态/店铺/逻辑组合、服务规则域防规则被反向弱化或编造）
- **RED 安全红线**：内部信息外泄/隐私/敏感词/违规引导/编造政策五条独立核查

**用户型产品特性设计**：模式判定与诚实兜底放行（工具返回空时忠实兜底+引导=产品预期行为，不扣分）；意图码驱动评测路径分流（某正向意图命中即 CTX 硬退出判通过）；卡片结构化数据盲区兜底（留空恒 N/A+"严禁假设可读取卡片数据"+"严禁因看不到而判 0"）。Rubric 以 Markdown+YAML 版本元数据存代码仓库，数十次迭代全有 changelog，在线 Prompt 直接从 Rubric 文件生成（文档与 Prompt 不两套）。框架统一+分域子项码（各域十几个子项）并存。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## Judge 模型与 Prompt 工程

模型选型：指令遵循+结构化输出优先（Judge 任务本质是"按 Rubric 做分类判定"不是生成）；轻量模型在多条件复合判定/边界 case/跨轮一致性上明显不如大模型——难度在"按规则做复杂判断"非语言理解；分阶段策略（先大模型+Prompt 验天花板并沉淀人标数据，后 SFT 蒸馏小模型降本）；temperature=0.1 梯度实验定值（0 会偶发僵死，高了同 case 多次结果不一致）。Prompt 幻觉压制/时间锚点（"明天到达"类表述 Judge 无法验证）/跨轮隔离防维度串味。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## 持续收敛闭环与归因

每轮迭代：抽样打标→Judge-Human 一致性校验→归因分析→优化 Prompt/Rubric→发车→重抽样；数科管"怎么评"（pipeline/Prompt/指标），测试产品管"评什么"（标准/严格程度）。评测归因可区分**数据层/Prompt 层/Rubric 层/Agent 真实质量**四类问题并驱动定点修复验证——核心结论：**评测可信度的瓶颈不在 Judge 模型本身，而在数据质量和评测标准的持续校准**。^[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12.md]

## 相关实体

- [[entities/meituan-turing-agent-evaluation-methodology-2026-08-06|美团图灵评测方法论]]（离线/受控环境分层评测，本文线上脏数据场景互补）
- [[entities/agent-eval-counterintuitive-insights-langfuse|Langfuse Agent 评测反直觉洞察]]
- [[entities/agent-skills-comprehensive-survey|Agent Skills 综述]]

→ [[raw/articles/taobao-shopping-assistant-online-eval-2026-10-12|原文存档]]
