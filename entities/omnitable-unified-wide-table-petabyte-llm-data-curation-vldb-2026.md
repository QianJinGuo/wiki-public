---
title: "OmniTable：PB 级 LLM 训练数据治理与探索的统一宽表系统"
created: 2026-09-04
updated: 2026-09-07
type: entity
tags: [llm-training, data-curation, data-engineering, data-preparation, wide-table, feature-lineage, fault-tolerance, vldb, ant-group, sft, metadata, catalog]
sources: [raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OmniTable：PB 级 LLM 训练数据治理与探索的统一宽表系统

> **核心主张**：OmniTable 用"逻辑统一、物理分离"的统一宽表系统管理 PB 级 LLM 训练数据（35+PB / 3050 亿条），把数据批次与特征列提升为"一等对象"，将物理表退回存储实现层，从而把数据准备从"逐表编排的工程苦役"变成"声明目标列的声明式任务"。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 问题：传统物理表范式的三项工程成本

大模型训练前，语料需经过解析、清洗、去重、质量评分、Token 化和样本组装。规模到 PB 级后，传统数据加工围绕物理表展开——每个数据源的解析/清洗/质量/标签/去重结果各落一张表，Web/代码/PDF/SFT 又各自维护一套流程。维护对象随源与特征持续膨胀：论文记录为补一个质量特征工程师需在任务画布上处理 **106 张表**。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

核心代价被概括为三项：**数据难定位**、**特征难回刷**、**结果难追溯**。表只保存结果而不记录结果如何算出，UDF 分散在不同代码库，输入列、算子版本、运行批次与下游训练任务间缺乏稳定关联。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 设计："逻辑统一、物理分离"

OmniTable 让数据批次和特征列成为一等对象，把物理表退回到存储实现层。逻辑层每行代表一条可追踪数据实体，每列保存某处理阶段状态或衍生特征（RawData/ProcessedData/TrainableData + 质量/领域/安全/去重特征列）。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

两类系统字段对齐列结构：**`__ai_unique_id_`** 全局主键（同一条数据跨来源/阶段/特征列用同一标识），**`__ai_append_name_`** 记录接入批次/来源/版本——数据回刷、点查和血缘追踪都有稳定锚点。生产按 Web/代码/PDF/post-SFT 划分四张领域逻辑宽表，最大 Web 宽表约 25PB/3000 亿条/800 逻辑列/200 已注册特征，受控实验可扩到 2500 列。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

逻辑列与物理位置的对应由 **Catalog** 保存。底层可拆行、拆列、合并小文件、调分区、建物化视图，上层 schema 与列语义不变。代价：热点列组物化约增加 8%–15% 存储开销。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 特征计算：从"画任务"到"报目标列"

特征注册时写明输入列、输出列、UDF/SQL/模型推理逻辑、版本与 CPU/GPU 偏好。工程师提交回刷只指定目标批次+目标特征，OmniTable 查询计算状态、沿列级依赖 DAG 找最小依赖闭包、按拓扑序生成物理执行计划。已完成结果复用，缺失祖先列进入计划，共享输入同引擎算子合并为单次扫描。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

任务成功后 Catalog 原子登记"批次—特征列"的状态/版本/物理位置/列级血缘，未提交结果不进稳定逻辑视图。工程师仍定义特征语义，系统接手依赖展开、执行路由、状态管理与结果提交。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 记录级容错：坏样本不再拖垮整批

非结构化语料混入异常编码/超长文本/损坏内容，PB 级数据即使 0.005% 异常也产生海量坏样本。传统批任务以任务为失败单位，一次 UDF OOM 或超时让多 TB 计算整体退出。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

OmniTable 把常见 UDF 故障隔离到**记录级**：每次 UDF 调用带超时与内存检查，遇 OOM/超时/未捕获异常记录样本 ID+异常类型+错误摘要，该条写 NULL，其余继续，错误进 error table。500GB/约 6 亿条特征任务中 31247 条异常（0.005%）：开启 failover 后 99.995% 记录一次处理完成、耗时约 6.2 小时零人工；关闭则任务直接失败；旧流程三轮排查约 52 小时（含 18 小时人工）。记录级包装约增 3%–5% 执行开销。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 算子融合与自适应调优：让 CPU/GPU 各司其职

特征计算形态差异大——文本长度/规则过滤适合 CPU/SQL，模型推理可跑 CPU 或 GPU。OmniTable 按用户声明、算子画像、引擎能力与集群负载，在 Spark、MaxCompute SQL 与 GPU 推理平台间选执行后端，并结合历史运行调资源。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

算子融合：8 个读 parsed_text 的 CPU/Spark 特征（约 2.5PB/3000 亿条），扫描从 8 次→1 次，CPU Hours 4.2 万→1.85 万（-55.9%），端到端 38h→14h（2.7×）。自适应调优针对 50GB/500GB/2TB 三规模首次提交成功率 100%，成本与专家手调差 ≤5%。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 后台治理：逻辑表变宽、物理布局持续演进

宽表上线后仍增批次与特征，小文件累积/分区倾斜/列数增长/查询热点变化拖慢访问。后台治理服务持续观察指标，自动执行小文件合并、行/列拆分、物化视图构建，用 **Prepare—Execute—Commit** 先准备新布局并物理重写、验证后原子切换 Catalog 映射，旧布局切换前继续服务、失败可回滚。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

列拆分让逻辑 schema 越过单引擎物理列数上限：2PB 测试逻辑列 200→2500，P95 延迟约 25s→38s，跨过约 1200 列物理上限；1TB–25PB 过滤导出吞吐稳定 18–23TB/h。单样本排查用全局 ID 索引把 ai_unique_id 定位到物理表/分区/row group，25PB 数据查完整逻辑行 P50 8.3s / P99 14.7s（全扫描 184s/612s+）。物化视图消除热点 JOIN：15 列/4 物理表场景过滤导出吞吐 4.8→20.1TB/h。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 端到端：5.6 倍提速

真实 SFT 数据准备任务（8 数据源、12 特征，9 CPU UDF + 3 GPU 推理）：旧流程约 2 天定位接入 + 9.5 天特征回填 + 2.5 天多表 JOIN 导出 = 约 14 天，含 45 手工步骤、24 条独立管道、35 张物理表。OmniTable 接入 0.5 天 + 回填 1.7 天 + 导出 0.3 天 = 约 2.5 天，手工步骤 12、独立命令 10。**端到端提速 5.6×，手工步骤 -73.3%，独立管道/脚本 -58.3%。**^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

时间省在：逐表编排变少、共享输入不再重复扫描、异常不频繁触发整批重跑、物理布局不必等性能下降后再人工调整。论文诚实标注成本（物化存储、记录级容错开销、治理集群资源），定位为 35+PB 生产检验的系统设计参考，核心思路是"先稳定逻辑语义，再让物理布局持续演进"。^[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026.md]

## 关联

- [[entities/sft-data-preparation-advanced-strategies-2026|SFT 数据准备进阶策略]]
- [[entities/从零训练steel-llm微调探索与评估|从零训练 Steel-LLM]]
- [[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition in LLM Pretraining]]
- [[entities/aws-aidl-paradigm-shift-platform-driven-data-engineering|阿里 AIDL 平台驱动数据工程]]
- [[entities/goodfire-predictive-data-debugging-post-training-anatomy-2026|GoodFire Predictive Data Debugging]]
- [[entities/llm-generative-retrieval-cq-sid-taobao-search-recall-2026|CQ-SID 生成式检索（同日 VLDB/检索体系落地）]]

→ [[raw/articles/omnitable-unified-wide-table-petabyte-llm-data-curation-vldb-2026|原文存档]]