---
title: 透镜提案队列
created: 2026-09-10
updated: 2026-09-13
type: query
tags: [meta, emergence, proposal-queue, dashboard]
confidence: high
---

# 透镜提案队列（Lens Proposal Queue）

> 透镜轮的产出从"一期特稿直接落页"改为"**提案卡 → 人审 → 落地**"的例行闭环——EvoScientist 的观察聚类→skill 提案→人工 `/autoskills` 审批模式，见 [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统透镜]] 观点五的工程解法。动机：涌现稿观点五的诊断（"wiki 缺的不是洞察的生产，是洞察的消费管线"）+ 8 篇涌现稿 [[queries/emergence-acceptance-gate|验收门]] 基线 8/8 REJECT 的教训——先小卡先审，比整稿落完再返工便宜。

## 流程

1. **落卡**：透镜轮/机制脚本（`ledger-backpropagate.py` 判定回传、`concept-gap-detector.py` 缺位检测）产出的补页候选，先在本页登记为卡片，不直接建页
2. **人审**：裁决 `approve` / `reject`（附一句理由）；卡是 LLM 起草、人拍板，与质量闭环三裁同纪律
3. **落地**：approve 后按**产出阶梯**选最廉价形态——wikilink 边 < comparison stub < 独立补页 < 涌现稿章节；落地后回填卡状态 `landed` 并链产物
4. **过期**：`proposed` 超过 60 天未裁决自动降级为 `stale`（防待办清单自身腐烂——8 月涌现稿已观察到一次）

提案卡不设独立文件，直接在下表追加行；复杂提案（需论证 >10 行）才开 drafts 存稿并在此处链入。

## 提案卡

| # | 提案 | 来源 | 预期产出（阶梯档位） | 状态 | 提出 | 裁决 |
|---|------|------|---------------------|------|------|------|
| 1 | 注入防御评测面轮换机制补页：payload 库的 held-out 锚点与轮换纪律（security 簇最大单向缺口） | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|跨簇迁移轮]]观点一空白 | 独立补页（`concepts/eval-surface-rotation`） | `landed`（concepts/eval-surface-rotation 已建） | 2026-09-10 | approve 2026-09-10 |
| 2 | [[concepts/agent-sandbox|沙箱]]四问框架增第五问："该策略保护哪类信息"（执行隔离 vs 信息隔离判别） | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|跨簇迁移轮]]半成功待办 | 已有页增量修改（最小档） | `landed`（第五问已加） | 2026-09-10 | approve 2026-09-10 |
| 3 | 断言动力学首测：12 月判定窗三件套（缺口复测 + 引力首测 + 台账 #2-#5 到期判定），含 `ledger-backpropagate.py` 回传收割 | [[queries/emergence-lens-rotation|度量轮]]决议 + 2026-09-10 仪器化 | 涌现稿章节 + 半衰期后验首算 | `scheduled`（2026-12，勿提前跑） | 2026-09-10 | — |
| 4 | supersedes 活边冷启动：机制进 SCHEMA/lint 但全库活边仅 1 条；时间切片透镜下一轮把已识别的术语替换链（karpathy v2/v3/v4 等）批量补 `superseded_by` 标注 | 2026-09-10 判定回传首跑发现 | wikilink 边批量补注（最小档） | `landed-partial`（karpathy v2/v3→v4 已标注；其余链需逐链方向判定） | 2026-09-10 | approve 2026-09-10 |
| 5 | 异源同题双页处置：RAG 双 slug 归并——[[concepts/retrieval-augmented-generation-rag|191 行有源版]]为主、[[concepts/rag-retrieval-augmented-generation|54 行无源版]] supersedes 或分工改写（双页互链为零，归并时补历史标注） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity|嵌入相似度轮]]观点一 | 已有页归并（增改档） | `landed`（分工而非归并：两页各有互补内容与独立 raw 源，互链+定位说明） | 2026-09-10 | approve 2026-09-10 |
| 6 | 异源同题双页处置：Claude 对比对双页——同日两篇对比文归并或分工（一版基准向、一版体验向），处置时补互链或 supersedes | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity|嵌入相似度轮]]观点一 | 已有页归并（增改档） | `landed`（分工：数据/选型向 vs 机制/体验向，互链+定位说明） | 2026-09-10 | approve 2026-09-10 |
| 7 | 同名 concept↔MOC 缺边家族补边：loop-engineering / agent-memory / openclaw / cloud-ai / ai-security 五对（涌现稿观点二表） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity|嵌入相似度轮]]观点二 | wikilink 边批量补注（最小档） | `landed`（5 对双向补边（1 对 MOC 侧已有）） | 2026-09-10 | approve 2026-09-10 |
| 8 | 草稿概念锚点补链：[[drafts/karpathy-2026-vibe-to-agentic-engineering|karpathy 草稿]]补 agentic/vibe 概念页与对比页锚点、三范式草稿补 hermes 概念页与 openclaw MOC | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity|嵌入相似度轮]]观点三 | wikilink 边补注（最小档） | `landed`（5 条概念/hub 锚点） | 2026-09-10 | approve 2026-09-10 |
| 9 | qmd 嵌入后端修复（基建）：Metal 着色器编译失败 + CPU 路径挂死 + cleanup 清走存量向量；临时旁路=Ollama nomic-embed-text 本地嵌入（本轮已验证） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-similarity|嵌入相似度轮]]撞库设置 | 脚本/skill 档修复 | `landed-partial`（Ollama 旁路固化（双脚本+manage.sh 注记）；上游 node-llama-cpp 修复待环境窗口） | 2026-09-10 | approve 2026-09-10 |
| 10 | redirect 墓碑治理：entities 层 1348 块墓碑（占层 32%），同文多拷贝留多墓碑（13 组）；不变式提案"一份归档一块桩"，多墓碑组合并、index 计页口径明确 | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault|全库下钻轮]]观点一 | 已有页归并（增改档） | `landed`（12 组 22 块重复桩归档+指路桩补全+9 页断链重定向；1 组活体重复转 #11） | 2026-09-10 | approve 2026-09-10 |
| 11 | 活体孪生分批归并：136 对（中英混合 slug 54 对优先——语言形态可确定性预筛；清单 `/tmp/emerge_live_twins.json` 需转存库内），处置=归并或补 supersedes/互链 | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault|全库下钻轮]]观点二 | 已有页归并（增改档，分批） | `landed`（批1：中英 54 对互链 + 清单入库 queries/embed-twin-inventory；批2（82 对）清单在库待逐对判） | 2026-09-10 | approve 2026-09-10 |
| 12 | concept↔entity 缺锚补链：644 对（阈值上），按分数分批；补链动作=概念页正文/关联节加实体锚 | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault|全库下钻轮]]观点三 | wikilink 边批量补注（最小档，分批） | `landed-b1`（批1 ≥0.85：219 锚/61 页；批2 0.80-0.85 待续） | 2026-09-10 | approve 2026-09-10 |
| 13 | "对比页须链被比对象"规则入 SCHEMA + 存量补链：118 对候选（高分段 85）；规则此前缺席，补链动作=对比页头部加被比概念/实体锚 | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault|全库下钻轮]]观点三 | SCHEMA 增订 + wikilink 边批量补注 | `landed`（SCHEMA 规则增订 + 批1 ≥0.80：33 锚/15 页） | 2026-09-10 | approve 2026-09-10 |
| 14 | loop-engineering MOC 收纳自家簇：5 个 loop-engineering 实体页均不链本簇 hub（同名 entity↔MOC 家族实体层实例） | [[drafts/wiki-emergent-viewpoints-2026-09-embed-fullvault|全库下钻轮]]观点三 | wikilink 边补注（最小档） | `landed`（5 实体入簇 hub） | 2026-09-10 | approve 2026-09-10 |
| 15 | "何时不要建记忆系统"概念页 + "何等信息不配写入"门槛清单：反方立场轮核心缺页（对照 when-not-to-harness-engineering 先例），门槛清单建议并入决策点 MOC | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial-memory|反方记忆轮]]论纲二 | 独立补页 + 已有 MOC 增改 | `landed`（concepts/when-not-to-memory-architecture 已建） | 2026-09-10 | approve 2026-09-10 |
| 16 | 记忆消融缺席入负结果观察区：全库零"同任务有/无记忆"对照记录，属缺席类负结果候选；补一条消融实验素材需求（非本库可产，需外部素材或自跑实验） | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial-memory|反方记忆轮]]论纲三 | [[queries/negative-results-registry|负结果登记簿]]观察区追加 | `landed`（观察区已登记） | 2026-09-10 | approve 2026-09-10 |
| 17 | 记忆簇立场光谱同框：五档立场（蒸馏教条→治理哲学→工程税→怀疑论→反方）散在五页从未同框，建议在决策点 MOC 增设立场光谱节收纳本稿光谱表 | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial-memory|反方记忆轮]]立场光谱节 | 已有 MOC 增改（最小档） | `landed`（光谱节入决策点 MOC） | 2026-09-10 | approve 2026-09-10 |
| 18 | 通道枚举判据页增订：第三簇（记忆）检验通过，增订"衍生通道"为一等通道类 + "写入前声明衍生预算"扩展陈述（涌现稿有现成扩展文本） | [[drafts/wiki-emergent-viewpoints-2026-09-crosscluster-memory|跨簇迁移第二对]]涌现点 | 已有页增订（增改档） | `landed`（判据页衍生通道节已增订） | 2026-09-10 | approve 2026-09-10 |
| 19 | 记忆影响谱系可观测性页候选："哪条偏好影响了这次决策"可回答性在库内零覆盖；与卡 #15 写入门槛清单互为表里（门槛管写入前、可观测管写入后） | [[drafts/wiki-emergent-viewpoints-2026-09-crosscluster-memory|跨簇迁移第二对]]空白 | 独立补页（`concepts/memory-derivation-observability`） | `landed`（concepts/memory-derivation-observability 已建） | 2026-09-10 | approve 2026-09-10 |
| 20 | 自校准认识仪器理论页：统一透镜轮的命名理论对象（五命题 + 真值衰减检索内核），作为第十一轮特稿一体先行落地 | [[drafts/wiki-emergent-viewpoints-2026-09-unification|统一轮]]观点一/二 | 独立补页（`concepts/self-calibrating-epistemic-instrument`） | `landed`（页已建；**裁决 pending**——自主研究轮先行落地，人审驳回即撤页留卡） | 2026-09-13 | — |
| 21 | SCEI 验证轮三处增订：理论页加"形式化"节（门控引理/删失估计引理/聚合形式约定）与"验证状态"滚动表；半衰期页机制类行加删失下界口径（Y=64 页日 → 95% 下界约两周）与真值钟取样规则 | [[drafts/wiki-emergent-viewpoints-2026-09-unification-verify|验证轮]]观点一/二/四 | 已有页增订（增改档 ×2） | `landed`（已改；**裁决 pending**——驳回即回滚两页增订） | 2026-09-13 | — |

## 与其他机制的关系

- **上游**：[[queries/emergence-lens-rotation|透镜轮换]]合约 v3 第 3 条——产出先落卡再落地；矛盾升级线与概念缺位检测器的候选照常走各自队列，仅补页动作在此汇合。
- **下游**：approve 的卡落地为页面后，涌现稿/补页过 [[queries/emergence-acceptance-gate|验收门]]（ZG 门 + lint）方可结单。
- **对账**：本页每月在 [[queries/vault-evolution-dashboard|进化仪表板]] 记一次队列深度与 landed 率——消费管线的健康指标。
