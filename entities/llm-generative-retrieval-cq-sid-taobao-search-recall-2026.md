---
title: "CQ-SID：LLM 生成式检索在电商搜索召回中的落地（类目约束 + Query-Item 对比 + EG-GRPO）"
created: 2026-09-04
updated: 2026-09-07
type: entity
tags: [generative-retrieval, cq-sid, semantic-id, rq-vae, ecommerce-search, recall, grpo, expert-guided-rl, llm-generation, taobao, alibaba, category-constrained]
sources: [raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04]
confidence: 0.88
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CQ-SID：LLM 生成式检索在电商搜索召回中的落地（类目约束 + Query-Item 对比 + EG-GRPO）

> 千问AI平台（朱建波/玄羿，阿里官方）CIKM'2026：把 LLM 生成式检索落地到手机天猫搜索召回补充路。核心是 CQ-SID（Category-and-Query Constrained Semantic ID）——基于 RQ-VAE 的语义簇 SID 构建 + 四步渐进式训练 + EG-GRPO 专家引导强化学习；线上 GMV+1.15%、UCTCVR+0.40%，已全量部署。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md]

## 定位：搜索召回的补充路，而非端到端生成

生成式检索把"索引-检索"流程参数化到单个 LLM（商品语义编码为离散 Semantic ID），LLM 自回归生成 SID 列表，把检索变纯生成任务。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md] 与快手 One 系列等推荐端到端生成不同，本文从电商搜索现实出发，把生成式检索定位于**召回阶段的一路补充**而非替代：搜索用户需求明确、对准确/相关要求极高，端到端模型在上亿商品池能否充分建模、避免马太效应、覆盖长尾仍是问题。

## 语义簇 SID：类目约束 + Query-Item 对比学习

CQ-SID 不追求"一物一 ID"，把 SID 视为**语义簇标识符**（语义/属性相似商品可映射同一 SID，不追求低碰撞率），满足语义聚合性、区分度保障、层次化结构三特性，在区分度与聚合性之间取得平衡。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md] 基于 RQ-VAE 引入两类约束：**类目引导残差量化**（三层码本 2048×1024×1024，第一级类目感知码本，商品数大于阈值的二级类目体系作约束，覆盖 1711 个有效类目；已知类目商品强制用类目标签作索引）与 **Query-Item 对比学习机制**；SID 后处理通过随机分割大簇防止单 SID 挂载过多商品、避免热门语义簇导致检索偏差，同时维持前缀层次化结构。

## 四步渐进式训练到个性化 Query2SID

用 Qwen2.5-0.5B 逐步学习文本到个性化 SID 生成：Step1 Item-SID（商品标题→SID 映射，理解语义结构与码本层次）；Step2 Query-SID（每个 Query 随机采样 N=3 个关联商品 SID 作目标，建立搜索词→SID）；Step3 (User+Query)-SID（加入性别/购买力/历史点击类目 SID 序列，按用户历史行为生成个性化 SID 列表）；Step4 用 GRPO 对齐精排。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md]

## EG-GRPO：稀疏奖励下对齐排序的专家引导强化学习

前三步以点击为目标 MLE 优化，Step4 用曝光商品集合作参考信号对齐精排（点击/曝光/购买商品 SID 集合算奖励，Svalid 为合法 SID 集合）。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md] 因点击购买远小于曝光、奖励稀疏，模型难获高质量信号，提出 **EG-GRPO（Expert-Guided GRPO）**：每批次生成 Group 中随机采样加入实际点击/曝光的 GroundTruth 作伪生成结果参与奖励计算与梯度更新，在稀疏奖励中获得稳定信号、避免探索偏离。

## 量化结果

- 语义召回（相同 Beam Size）CQ-SID beam@1/10/100 Hitrate 0.0758/0.3161/0.6181，较 RQ-VAE 提升 +26.76%/+22.57%/+18.89%；消融证明类目约束与对比学习协同。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md]
- Top-1K 截断：CQ-SID 仅需 beam=30 达 0.4422 vs RQ-VAE beam=65 达 0.4275（Beam Size 减 53.85%、Hitrate 提升 3.44%）；个性化场景 CQ-SID beam=160 达 0.7984 vs RQ-VAE beam=195 达 0.7607（Beam 减 17.95%、Hitrate 提升 4.96%）。更小 Beam Size = 更低延迟与资源消耗同时更高召回质量。
- 对齐排序：仅标准 GRPO（K=0）因稀疏奖励模式集中效应（概率集中少数高置信 SID、降深层 beam 多样性）使 beam@10 点击 Hitrate 反降；EG-GRPO(K=2/4) 点击/曝光 Hitrate 及曝光覆盖度一致提升，实现精确性与多样性平衡（多目标帕累托改进）。
- 线上：动态 BeamSize=[20,50,100]、平均 RT 约 40ms，**GMV+1.15%、UCTCVR+0.40%**，已全量部署；生成式召回路带重曝光 50.25%、点击 58.96%、成交 72.63%。

## 验证边界与展望

本文是真实业务数据 + 线上生产部署的完整落地，但底池从全量按效率筛选约 2100 万商品作初始生成式底池（假设性收敛）、且定位为补充路而非端到端。^[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04.md] 未来探索搜索生成式"召粗一体"范式减少漏斗损失，逐步验证端到端生成在垂类/长尾流量场景的可行性边界。

## 与现有生成式检索知识的关系

- **互补（同为 RQ-VAE 语义 ID 生成式检索，不同系统/场景）**：[[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索工业实践]]（4 层 4096 码本、20 亿商品、广告召回+排序）聚焦"一物一 ID"碰撞率最低化；CQ-SID 反其道追求**语义簇**、加类目约束与 Query-Item 对比、用 EG-GRPO 对齐电商搜索排序——两条路线在"区分度 vs 聚合度"上取相反立场，互为对照。
- **互补（偏好对齐）**：[[entities/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析|RAD-DPO 生成式检索偏好对齐]]（SIGIR 2026）用 DPO 做偏好对齐；本文用 GRPO 变体（EG-GRPO，注入 GT）稀疏奖励对齐，RL 对齐策略不同。
- 与传统检索的对比呼应 [[entities/agent-vs-workflow-control-continuum-framework]] 以外检索范式的演进；电商搜索召回的多路补充设计可对照百科内其他搜索/召回实体。

→ [[raw/articles/cq-sid-generative-retrieval-taobao-search-recall-2026-09-04|原文存档]]