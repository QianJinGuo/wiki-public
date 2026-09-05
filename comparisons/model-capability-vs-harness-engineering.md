---
title: 模型能力 vs Harness Engineering：两论点对抗
created: 2026-09-05
updated: 2026-09-05
type: comparison
tags: [harness-engineering, model-capability, debate, agent, evaluation]
confidence: 0.75
provenance_state: merged
contradicted_by: []
---

# 模型能力 vs Harness Engineering：两论点对抗

> 反方立场透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-adversarial|2026-09 涌现稿]]）产物。此前 [[entities/harness-engineering|Harness Engineering 主页]]的反方标注承认"反方观点长期无独立页面，由规范主页收留双方论据"——本页把收留变成正式对峙。

## 正方：Harness 是第三代工程范式的承重层

**主张**：模型概率性输出不可直接用于生产，可靠性的增量来自 harness（上下文管理、反馈回路、工具合约、评测防火墙），而非等待更强模型。

**库内证据**：[[concepts/harness-engineering-framework|框架页]]的 5 项普适原则（上下文胜指令/规划执行分离/反馈回路/一次一事/代码库即文档）；[[entities/harness-engineering|范式主页]]的学科时间线（90 天确立）；PKU 论文的工件分类（AGENT.md/会话初始化/Sprint 合约）。生产侧案例：快手交付管线、淘宝商品域 agent 架构等 `harness` 标签簇 517 页中的重实践记录。

**最强形式**：MirrorCode 长程基准（[[entities/mirrorcode-long-horizon-benchmark-epoch-ai-metr|14 小时重写 2-17 周人类工作量]]）显示的正是"模型能力 × 执行脚手架"的复合产出——没有脚手架的长程任务，当前模型独立完成率急剧下降。

## 反方：瓶颈在模型，harness 是对模型短板的过拟合

**主张**：harness 解决的每个问题（幻觉、格式漂移、长上下文遗忘）都是模型缺陷的影子；为影子建工程，是在缺陷上资本化。

**库内证据**：
1. **代际侵蚀**：框架页自己记录 Opus 4.5→4.6 使 harness 成本节省 38%——模型每前进一代，脚手架价值就塌一批（详见 [[concepts/when-not-to-harness-engineering|边界条件一]]）。
2. **强模型上的负收益**：[[entities/gene-gep-evomap-qinghua-strategy-genes-arxiv-2604-15097-2026|Gene-GEP]] 在 Pro 上 Skill -9.4；[[entities/computer-use-45x-more-expensive-than-structured-apis|结构化 API 便宜 45 倍]]。
3. **第一性立场的长期缺席**：反方在库内长期"无独立页面"（主页标注自认），说明簇的论证结构是单向的——直到本页。

**最强形式**：若 METR 式自主时长曲线按当前斜率继续（台账 #3 `on-track`），三到四代模型之内，今天 harness 承重的场景会被裸模型直接覆盖——那时 harness 工程的全部存量资产一次性贬值。

## 裁决框架（可直接检验的三问）

| 检验 | 正方赢 | 反方赢 |
|------|--------|--------|
| 长程任务上裸模型与 harness-agent 的差距 | 持续 ≥2 代模型不收敛 | 一代内收敛 ≥50% |
| harness 工件平均存活期 | 随模型升级**变长** | 变短（衰减假说成立） |
| 同任务 harness 开销占比 | 占比下降但绝对收益上升 | 绝对收益同步下降 |

按 [[queries/prediction-ledger|台账]]口径，这三问均可登记为带到期日的可判定预测。

## 综合判断（暂裁）

库内证据支持的不是二选一，是**分层时序**：模型能力决定 harness 的**必要总量**（逐代收缩），harness 工程决定**当前代际的可用地板**（逐代抬高）。错误姿态有两个：用上一代的 harness 约束这一代的模型（过拟合，Gene-GEP 案例），以及用下一代的模型预期取消这一代的 harness（裸奔，长程任务崩溃案例）。运维姿态见 [[concepts/when-not-to-harness-engineering|反向检查清单]]。

## 检索入口

正方：[[concepts/harness-engineering-framework]] · [[moc/coding-agent-practice]] ｜ 反方：[[concepts/when-not-to-harness-engineering]] · [[queries/negative-results-registry]] ｜ 中立机制：[[concepts/claim-half-life]] · [[concepts/eval-optimizer-firewall]]
