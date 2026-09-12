---
title: 涌现透镜轮换
created: 2026-09-05
updated: 2026-09-13
type: query
tags: [meta, emergence, synthesis, dashboard]
confidence: high
---

# 涌现透镜轮换（Emergence Lens Rotation）

> 目的：把"透镜撞库"从偶发事件变成周期仪式。全库最深的两个产物——[[drafts/wiki-emergent-viewpoints-2026-07|2026-07 结构信号透镜]]与[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统透镜]]——都是同一方法的产物：**拿一个透镜去撞存量，记录哪里撞出火花、哪里撞出矛盾、哪里两边都是空的**。本页是透镜清单与轮换状态，供 [[queries/vault-evolution-dashboard|进化仪表板]] 的 L4 原理层调度。

## 方法合约（每轮固定）— v3（2026-09-10 起）

1. **选一个透镜**（不混用，一轮只透一个）
2. **撞库**：透镜 × 相关知识簇，重点记录三类发现——火花（新连接）、矛盾（页面间未对话的对立立场）、空白（两边都没有）
3. **定额产出**：1 篇 `drafts/wiki-emergent-viewpoints-YYYY-MM-<透镜名>`；补页候选**先落 [[queries/lens-proposals|提案队列]]为提案卡，人审 approve 后再落地**（v3：特稿照出，补页走 EvoScientist 闭环）
4. **验收**：涌现稿必须至少包含一条"本 wiki 此前不存在 X 页面/X 一等轴"的陈述并补齐；**`node scripts/emergence-gate.mjs --strict` 通过**（v3：ZG 门硬约束，协议见 [[queries/emergence-acceptance-gate|验收门]]）；lint 0 error
5. **登记**：在下方轮换状态表记录日期与产量，log.md 留痕

> 合约版本：v1（2026-09-05 定型五条）→ v2（度量轮免定额、节奏分化）→ v3（补页走提案卡闭环 + ZG 验收门，2026-09-10）。

## 透镜目录（8 个，按启动成本排序）

| # | 透镜 | 撞什么 | 预期火花 | 启动成本 |
|---|------|--------|----------|----------|
| 1 | **时间切片** | "2026-Q1 的判断，Q3 还成立吗"：取 90 天前同主题页 vs 近 90 天页 | 过时结论、被推翻的预测、 supersedes 链 | 低（纯存量） |
| 2 | **概念缺位** | 链接图：in-link 密集但无 concept hub 收拢的主题；跨 ≥5 entity 的窄标签簇 | 候选概念页清单（喂 [[moc/wiki-pending-concepts-roadmap]]） | 低 |
| 3 | **反方立场** | 让"最强批评者"读一个 MOC，写出 steelman 反方 | 每个 hub 页的立场光谱、缺失的"何时不要 X"页 | 低 |
| 4 | **跨簇迁移** | A 簇机制搬到 B 簇（如评测防火墙 → security / memory） | "X 簇的 Y 机制在 Z 场景是否成立"的 comparison 页 | 中 |
| 5 | **预测到期** | 带时间谓词的断言（"X 将在 Y 前取代 Z"）集中回访 | prediction ledger + 对/错判定（时间透镜的常量化） | 中 |
| 6 | **嵌入相似度** | qmd 嵌入找"语义近但无互链"的页对 | 该连未连的边、误判的重复页 | 中（需跑 qmd） |
| 7 | **外部系统** | 外部 repo/产品合集当探针（[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|phd-lens 模式]]） | 一等设计轴（如"经验抽象度"）、wiki 缺的整页概念 | 高（需外部素材） |
| 8 | **统一** | 透镜制自身产出当撞库对象：多轮机制/度量/矛盾收拢为单一可证伪理论 | 命名理论对象 + 台账预测条目 | 低（纯存量，但要求机制页已存在） |

## 轮换状态

| 日期 | 透镜 | 产出 | 备注 |
|------|------|------|------|
| 2026-07 | 结构信号（前身） | [[drafts/wiki-emergent-viewpoints-2026-07]] | 透镜制之前，方法验证轮 |
| 2026-08-29 | 外部系统（phd） | [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens]] + [[concepts/eval-optimizer-firewall]] 等 | phd 七仓库透镜 |
| 2026-09-05 | **① 时间切片** | [[drafts/wiki-emergent-viewpoints-2026-09-time-slice]] + [[concepts/claim-half-life]] + [[queries/prediction-ledger]]（种子 7 条） | 172 条时间断言撞库；首测半衰期：术语 30d / 代际 59d / 发布预期 109d；supersedes 空转定性 |
| 2026-09-05 | **② 概念缺位** | [[drafts/wiki-emergent-viewpoints-2026-09-concept-gap]] + [[concepts/world-models]] + [[concepts/agent-sandbox]] + 工具 concept-gap-detector.py | D1:4/D2:229/D3:133/D4:106；伪 hub 与跨目录孪生、标签-概念断线两结构发现；台账中期检视附带完成 |
| 2026-09-05 | **③ 反方立场** | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial]] + [[concepts/when-not-to-harness-engineering]] + [[comparisons/model-capability-vs-harness-engineering]] + [[queries/negative-results-registry]] | harness 簇检验：厂商指控被驳回（1%）、同温层 49%、衰减推论缺席；5 条负结果入册；跨目录孪生同步清理 |
| 2026-09-05 | **④ 跨簇迁移** | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster]] + [[concepts/channel-enumeration-criterion]] + [[comparisons/information-isolation-vs-execution-isolation]] | 评测防火墙 × 沙箱：通道枚举判据合成；判分 2 成功 1 半成功 1 空白；对抗页三问登记台账 #8-#10 |
| 2026-09-05 | **⑤ 概念缺位·复测（度量轮）** | [[drafts/wiki-emergent-viewpoints-2026-09-metrology]] + 检测器升级（delta/history/D2 排除）+ 仪表板六指标基线 | D3 133→128（覆盖 5：实覆盖 2/重标签 2/借道 1）；新增缺口 0；合约 v2（度量轮免定额）；节奏分化决议 |
| 2026-09-10（补登） | 第六轮·**外部系统**（目录⑦） | [[drafts/wiki-emergent-viewpoints-2026-09-external-probe]] | ECC 蒸馏集与商业 Agent 产物撞库：簇间独立收敛同原语集、SOUL.md 身份可移植工件、厂商技能格式空白；此前漏登轮换表，随本轮补记 |
| 2026-09-10 | 第七轮·**嵌入相似度**（目录⑥） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity]] + 提案卡 #5-#9 | 综合层 284 页全量嵌入（Ollama nomic-embed 旁路，qmd 后端故障）；三类发现：异源同题双页（RAG 与 Claude 对比对）、同名 concept↔MOC 缺边家族、草稿层概念锚点薄层；结构性豁免刻画谓词精度边界；合约 v3 首轮（产出先落卡、验收门 PASS） |
| 2026-09-10 | 第八轮·**嵌入相似度·全库下钻**（目录⑥，同透镜第二层） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault]] + 提案卡 #10-#14 | entities 层 4208 页首挖、全库 4546 页嵌入 → 候选 18847 对；四类新发现：redirect 墓碑积压（1348 块占层三分之一，"一份归档一块桩"不变式缺席）、活体孪生 136 对（中英双 slug 同文为主型，slug 语言形态确定性预筛）、concept↔entity 缺锚 644 对、comparison↔concept 缺锚 118 对（SCHEMA 规则缺席）；类型过滤防墓碑假阳性为方法论增量；**透镜⑥判定收敛**（残差清单随卡交付） |
| 2026-09-10 | 第九轮·**反方立场·记忆簇**（目录③第二轮） | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial-memory]] + 提案卡 #15-#17 + 台账 #11 | steelman 三论纲：记忆是正在消失的问题（跨代贬值可证伪化→台账 #11 预写口径）、记忆即攻击面（SkillJack/bleeding-LLAMA/清算不可行，写入门槛清单缺位）、胜者形态是纯文本+模型理解力（记忆消融全库零记录→负结果候选）；五档立场光谱首次同框；补页走卡 |
| 2026-09-10 | 第十轮·**跨簇迁移·第二对**（目录④第二轮） | [[drafts/wiki-emergent-viewpoints-2026-09-crosscluster-memory]] + 提案卡 #18-#19 | 通道枚举判据第三簇（记忆）检验：判分成功（bleeding-LLAMA 存储通道）/扩展（SkillJack 衍生链——通道清单=I/O+衍生，写入前声明衍生预算）/空白（影响谱系可观测性零覆盖）；合成与检验分离两轮，透镜④已备料对子清空 |
| 2026-09-13 | 第十一轮·**统一**（目录⑧新增） | [[drafts/wiki-emergent-viewpoints-2026-09-unification]] + [[concepts/self-calibrating-epistemic-instrument]] + 台账 #12-#14 + 外部摄入 [[raw/articles/arxiv-2509-19376-temporal-rag-freshness-trend-detection]] | 八机制首次合龙为命名理论：自校准认识仪器（SCEI 五命题）+ 真值衰减检索内核（score = sim × 生存 × 权威）；外部查新：temporal RAG 半衰期手工定值 + "学习型时序模型"列为 future work = 公开留白恰在本库开工处；理论页随特稿落地，提案卡 #20 裁决 pending |
| 2026-09-13 | 第十二轮·**统一·理论验证**（目录⑧第二层） | [[drafts/wiki-emergent-viewpoints-2026-09-unification-verify]] + 工具 `tdr-retrospective-eval.py` + 提案卡 #21 | 验证轮：门控引理 + 删失估计引理入册、聚合形式辨析（先检查后立论：原稿无混写）；机制类"≈不衰变"被删失下界降级（Y=64 页日 → 95% 下界约两周，随暴露量机械收紧）；TDR 回溯评测四案例——纯相似度术语链错序被修、**全局手调衰减对照组误伤机制页而 TDR 免疫（判别性实验）**；复访率归一削弱幸存者偏差质疑；反方五问（问三被实验+引理联合驳回，问五接纳为开放问题）；未消耗 12 月判定窗 |
| — | （待跑：内容轮——反方立场换簇或跨簇迁移第二对；年末判定窗按提案卡 #3 锁定勿提前） | | 下一轮占位 |

## 与其他机制的关系

- **矛盾升级线**（[[queries/vault-evolution-dashboard|见仪表板]]）是 L3 张力层的常驻管道；透镜轮是 L4 原理层的周期脉冲。四裁判为 `debate` 的对子可以直接成为透镜轮的素材。
- 透镜轮属于 WORKFLOW Phase 3 Evolve 的 **Frontier Track** 实例："vault 缺的不是更多笔记而是更好的研究议程"。
- **v3 仪器化**（2026-09-10）：补页消费走 [[queries/lens-proposals|提案队列]]，产出质检走 [[queries/emergence-acceptance-gate|验收门]]，时间断言回访走 [[queries/prediction-ledger|台账]] + `ledger-backpropagate.py` 判定回传——透镜回答"哪里有火花"，仪器回答"火花是不是真的"。
- L5 输出取材：月报/周报选题优先从最近的涌现稿与 debate 页取，形成"摄入→张力→原理→叙事"的完整梯度。
